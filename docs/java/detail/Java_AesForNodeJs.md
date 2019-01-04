### java

```java
package com.ibingbo.util;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.security.MessageDigest;

/**
 * Created by bing on 2017/2/14.
 */
public class AESForNodejs {

    public static final String DEFAULT_CODING = "utf-8";


    public static String decrypt(String encrypted, String seed) throws Exception {
        byte[] keyb = seed.getBytes(DEFAULT_CODING);
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] thedigest = md.digest(keyb);
        SecretKeySpec skey = new SecretKeySpec(thedigest, "AES");
        Cipher cipher = Cipher.getInstance("AES");
        cipher.init(Cipher.DECRYPT_MODE,skey);

        byte[] clearbyte = cipher.doFinal(toByte(encrypted));
        return new String(clearbyte);
    }

    public static String encrypt(String content, String key) throws Exception {
        byte[] input = content.getBytes(DEFAULT_CODING);

        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] thedigest = md.digest(key.getBytes(DEFAULT_CODING));
        SecretKeySpec skc = new SecretKeySpec(thedigest, "AES");
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, skc);

        byte[] cipherText = new byte[cipher.getOutputSize(input.length)];
        int ctLength = cipher.update(input, 0, input.length, cipherText, 0);
        cipher.doFinal(cipherText, ctLength);

        return parseByte2HexStr(cipherText);
    }
    /**
     * 字符串转字节数组
     * @param hexString
     * @return
     */
    private static byte[] toByte(String hexString) {
        int len = hexString.length() / 2;
        byte[] result = new byte[len];
        for (int i = 0; i < len; i++) {
            result[i] = Integer.valueOf(hexString.substring(2 * i, 2 * i + 2), 16).byteValue();
        }
        return result;
    }

    /**
     * 字节转16进制数组
     * @param buf
     * @return
     */
    private static String parseByte2HexStr(byte buf[]) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < buf.length; i++) {
            String hex = Integer.toHexString(buf[i] & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            sb.append(hex);
        }
        return sb.toString();
    }

    public static void main(String[] args) throws Exception {
        System.out.println(AESForNodejs.encrypt("java编程思想", "1234fghjnmlkiuhA"));
        System.out.println(AESForNodejs.decrypt("e877eed92cfbc86083fab4ec3cf0b8e561f7a4e112c3e1fd67fa26a7e331da90", "1234fghjnmlkiuhA"));
    }
}
```

### node

```javascript
/** 
 * aes加密 
 * @param data 
 * @param secretKey 
 */  
encryptUtils.aesEncrypt = function(data, secretKey) {  
    var cipher = crypto.createCipher('aes-128-ecb',secretKey);  
    return cipher.update(data,'utf8','hex') + cipher.final('hex');  
}  
  
/** 
 * aes解密 
 * @param data 
 * @param secretKey 
 * @returns {*} 
 */  
encryptUtils.aesDecrypt = function(data, secretKey) {  
    var cipher = crypto.createDecipher('aes-128-ecb',secretKey);  
    return cipher.update(data,'hex','utf8') + cipher.final('utf8');  
} 
```