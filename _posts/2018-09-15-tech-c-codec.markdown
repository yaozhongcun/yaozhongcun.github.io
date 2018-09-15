---
layout: post
title:  "C++ utf8转unicode"
date:   2018-09-11 08:14:12 +0800
categories: 技术 
---

提问: 用户输入为utf8编码的字符串, 如下两个问题如何解决:
1. 用户输入中中文字符的数量; 
2. 屏蔽字库中不支持的emoji字符. 

utf8作为一个变长的字符编码, 不利于字符的比较(有的时候3字节,有的时候4字节).
字符编码规则,一般只按unicode给出区段.
所以一个通用的解决方案是, 将utf8转为unicode,数符合规范的unicode字符的个数或者比较相应的区段.


## utf8和unicode编码介绍

标准的unicode采用4字节的整形来表示一个字符., 我们成为UCS4(通用字符集, 4字节)
其中又分为了16个平面, 0号平面,我们也可以用两个字节的整形来表示一个字符, 就是最开始的UCS2
* 平面  始末字符值   中文名称    英文名称
* 0号平面    U+0000 - U+FFFF 基本多文种平面 Basic Multilingual Plane，简称BMP
* 1号平面    U+10000 - U+1FFFF   多文种补充平面 Supplementary Multilingual Plane，简称SMP
* 2号平面    U+20000 - U+2FFFF   表意文字补充平面    Supplementary Ideographic Plane，简称SIP
* 3号平面    U+30000 - U+3FFFF   表意文字第三平面（未正式使用[1]）  Tertiary Ideographic Plane，简称TIP
* 4号平面
* 至
* 13号平面   U+40000 - U+DFFFF   （尚未使用）
* 14号平面   U+E0000 - U+EFFFF   特别用途补充平面    Supplementary Special-purpose Plane，简称SSP
* 15号平面   U+F0000 - U+FFFFF   保留作为私人使用区（A区）[2]    Private Use Area-A，简称PUA-A
* 16号平面   U+100000 - U+10FFFF 保留作为私人使用区（B区）[2]    Private Use Area-B，简称PUA-B
 
utf8虽然最多使用6字节, 但就现有的unicode平面来讲, 最长也只用了4个字节.
* Unicode 和 UTF-8 之间的转换关系表 ( x 字符表示码点占据的位 )
* 码点的位数   码点起值    码点终值    字节序列    Byte 1  Byte 2  Byte 3  Byte 4  Byte 5  Byte 6
*  7 U+0000  U+007F  1   0xxxxxxx
* 11  U+0080  U+07FF  2   110xxxxx    10xxxxxx
* 16  U+0800  U+FFFF  3   1110xxxx    10xxxxxx    10xxxxxx
* 21  U+10000 U+1FFFFF    4   11110xxx    10xxxxxx    10xxxxxx    10xxxxxx
* 26  U+200000    U+3FFFFFF   5   111110xx    10xxxxxx    10xxxxxx    10xxxxxx    10xxxxxx
* 31  U+4000000   U+7FFFFFFF  6   1111110x    10xxxxxx    10xxxxxx    10xxxxxx    10xxxxxx    10xxxxxx


## 实现 
使用Utf8ToUcs4函数, 可以将utf8字符串转成vector<uint32_t>数组, 数组中的元素,uint32的整形对应一个unicode字符的编码.  然后使用该数组与做后续处理即可(和目标字符编码比较,或者计算字符数量)
```  c++
size_t Utf8StrTool::Utf8CharToUcs4Char(const std::string& utf8_str, size_t cursor, uint32_t& ucs4) {
    //We do math, that relies on unsigned data types
    const unsigned char* utf8TokUs = reinterpret_cast<const unsigned char*>(&(utf8_str[cursor]));

    //Initialize return values for 'return false' cases.
    ucs4 = 0;

    //Decode
    if (0x80 > utf8TokUs[0]) {
        //Tokensize: 1 byte
        ucs4 = static_cast<const uint32_t>(utf8TokUs[0]);
        return 1;
    } else if (0xC0 == (utf8TokUs[0] & 0xE0)) {  // 0xC0 11000000, 0xE0 11100000
        //Tokensize: 2 bytes
        if ( 0x80 != (utf8TokUs[1] & 0xC0) ) {
            return 0;
        }
        ucs4 = static_cast<const uint32_t>(
                  (utf8TokUs[0] & 0x1F) << 6
                | (utf8TokUs[1] & 0x3F)
            );
        return 2;
    } else if (0xE0 == (utf8TokUs[0] & 0xF0)) {
        //Tokensize: 3 bytes
        if (   ( 0x80 != (utf8TokUs[1] & 0xC0) )
            || ( 0x80 != (utf8TokUs[2] & 0xC0) )
           ) {
            return 0;
        }

        ucs4 = static_cast<const uint32_t>(
                  (utf8TokUs[0] & 0x0F) << 12
                | (utf8TokUs[1] & 0x3F) << 6
                | (utf8TokUs[2] & 0x3F)
            );
        return 3;
    } else if (0xF0 == (utf8TokUs[0] & 0xF8)) {
        //Tokensize: 4 bytes
        if (   ( 0x80 != (utf8TokUs[1] & 0xC0) )
            || ( 0x80 != (utf8TokUs[2] & 0xC0) )
            || ( 0x80 != (utf8TokUs[3] & 0xC0) )
           ) {
            return 0;
        }

        ucs4 = static_cast<const uint32_t>(
                  (utf8TokUs[0] & 0x7) << 18
                | (utf8TokUs[1] & 0x3F) << 12
                | (utf8TokUs[2] & 0x3F) << 6
                | (utf8TokUs[3] & 0x3F)
            );
        return 4;
    }  // 其他位面不支持

    return 0;
}

bool Utf8StrTool::Utf8ToUcs4(const std::string& utf8_str, std::vector<uint32_t>& output) {
    auto len = utf8_str.size();
    size_t cursor = 0;
    static const size_t MB_LEN = 1024*1024;
    if (len > MB_LEN ) {
        return false;
    }

    while (cursor <  len) {
        uint32_t ucs4 = 0;
        auto utf8_token_len = Utf8CharToUcs4Char(utf8_str, cursor, ucs4);
        output.push_back(ucs4);
        cursor += utf8_token_len;
        if ( utf8_token_len <= 0) {
            return false;
        }

    }

    return true;
}

static const std::vector< std::pair<uint32_t, uint32_t> > EMOJI_CHAR_RANGE_VEC = {
                {0x2190, 0x21FF},
                {0x2600, 0x26FF},
                {0x2700, 0x27BF},
                {0x3000, 0x303F},
                {0x1F300, 0x1F64F},
                {0x1F680, 0x1F6FF}
};

```
