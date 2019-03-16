---
title: shell脚本调用阿里云RAM用户API接口
date: 2019-03-13 14:18:21
categories:
 - other
tags:
 - shell
---

此脚本为调用阿里云RAM 接口，以ListUsers 为例，需要准备AS并赋权。<!--more-->

```bash
#!/bin/bash
# 接口调用脚本，需要添加自定义参数

RAM_server="https://ram.aliyuncs.com/?"
AS="xxxxxxxxxxxxxxxxxxxxxxxxx&"
AccessKeyId="xxxxxxxxxxxxxxxxxxxxx"
Action="ListUsers"   #自定义参数
Format="json"
SignatureMethod="HMAC-SHA1"
#SignatureNonce=$(date +%s%N | md5sum | head -c 10)
SignatureNonce=`cat /proc/sys/kernel/random/uuid`
Transport="true"
SignatureVersion="1.0"
Timestamp=$(date -u "+%Y-%m-%dT%H:%M:%SZ")
Version="2015-05-01"

#	 Format=json
#        &Version=2015-05-01
#        &Signature=Pc5WB8gokVn0xfeu%2FZV%2BiNM1dgI%3D
#        &SignatureMethod=HMAC-SHA1
#        &SignatureNonce=15215528852396
#        &SignatureVersion=1.0
#        &AccessKeyId=key-test
#        &Timestamp=2012-06-01T12:00:00Z
#编码方法

function encode()
{
    encoded_str=`echo "$*" | awk 'BEGIN {
        split ("1 2 3 4 5 6 7 8 9 A B C D E F", hextab, " ")
        hextab [0] = 0
        for (i=1; i<=255; ++i) {
            ord [ sprintf ("%c", i) "" ] = i + 0
        }
    }
    {
        encoded = ""
        for (i=1; i<=length($0); ++i) {
            c = substr ($0, i, 1)
            if ( c ~ /[a-zA-Z0-9.-]/ ) {
                encoded = encoded c             # safe character
            } else if ( c == " " ) {
                encoded = encoded "+"   # special handling
            } else {
                # unsafe character, encode it as a two-digit hex-number
                lo = ord [c] % 16
                hi = int (ord [c] / 16);
                encoded = encoded "%" hextab [hi] hextab [lo]
            }
        }
        print encoded
    }' 2>/dev/null`
}

encode $pa
p=$encoded_str


#SourceIp=42.120.74.222
#SecureTransport=true
# ListUsers

encode $AccessKeyId
AccessKeyId=$encoded_str

encode $Action
Action=$encoded_str


encode $Format
Format=$encoded_str

encode $SignatureMethod
SignatureMethod=$encoded_str


encode $SignatureNonce
SignatureNonce=$encoded_str


encode $SignatureVersion
SignatureVersion=$encoded_str

encode $Timestamp
Timestamp=$encoded_str

encode $Version
Version=$encoded_str



#接口公共参数：Format Version AccessKeyId Signature SignatureMethod SignatureVersion SignatureNonce Timestamp
#对数组字典进行排序
#string_Array=(AccessKeyId Action Format SignatureMethod SignatureNonce SignatureVersion Timestamp Version)
string_Array=(Format Version AccessKeyId Action SignatureMethod SignatureVersion SignatureNonce Timestamp Transport)
sorted=($(printf '%s\n' "${string_Array[@]}"|sort))
#echo ${sorted[*]}

String=""

for ((i=0;i<${#sorted[*]};i++)) do
#String=$String"&${sorted[i]}=""\$"${sorted[i]}
String=$String"&${sorted[i]}="`eval echo '$'"${sorted[i]}"`
#echo ${sorted[$i]}
done

#echo $String
#String_awk={$String:1:${#String}}
#截取字符串，把首部&去掉
String_awk=`expr substr "$String" 2 ${#String}`
echo $String_awk

#canonicalizedQueryString="AccessKeyId="$AccessKeyId"&Action="$Action"&Format="$Format"&SignatureMethod="$SignatureMethod"&SignatureNonce="$SignatureNonce"&SignatureVersion="$SignatureVersion"&Timestamp="$Timestamp"&Version="$Version
#echo $canonicalizedQueryString
canonicalizedQueryString=$String_awk
#echo $canonicalizedQueryString

#再对上边的string进行url编码
encode $canonicalizedQueryString
canonicalizedQueryStringEncode=$encoded_str

#签名的字符串拼接 get&/&
StringToSign="GET&%2F&"$canonicalizedQueryStringEncode

#签名，根据sha1-hmac 然后base64转码
signature=$(echo -n $StringToSign | openssl sha1 -hmac "$AS" -binary | base64)
#signature=$(echo -n $StringToSign | openssl dgst -sha1 -mac HMAC -macopt key:$AS -binary | base64)
echo -e "签名："$signature \

#解决bug，访问接口有时失败的问题
encode $signature
signature=$encoded_str

#把签名值最后加上，拼接请求体
http_body=$canonicalizedQueryString"&Signature="$signature

#sleep 3     # curl请求
curl -X GET $RAM_server$http_body -H "accept: */*" && echo  ""

```
