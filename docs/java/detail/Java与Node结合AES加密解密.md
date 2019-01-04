## Node加密解密

```node
var crypto = require("crypto");
/**
 * aes-128-cbc对称加密
 * @param {String} data，加密的明文
 * @param {String} secretKey，密钥
 * @param {Buffer} iv，向量，16位字节数组
 * @return {String} 密文
 * @api public
 * @remark 密钥的生成规则：普通字符串，用UTF8转换成Buffer后，需要对此计算MD5，再转换成Buffer对象。
 */
function cbcEncrypt(data, secretKey, iv) {
 secretKey = new Buffer(secretKey, "utf8");
 secretKey = crypto.createHash("md5").update(secretKey).digest("hex");
 secretKey = new Buffer(secretKey, "hex");
 var cipher = crypto.createCipheriv("aes-128-cbc", secretKey, iv), coder = [];
 coder.push(cipher.update(data, "utf8", "hex"));
 coder.push(cipher.final("hex"));
 return coder.join("");
}
 
/**
 * aes-128-ecb对称解密
 * @param {String} data，解密的密文
 * @param {String} secretKey，密钥
 * @param {Buffer} iv，向量，16位字节数组
 * @return {String} 明文
 * @api public
 */
function cbcDecrypt(data, secretKey, iv) {
 secretKey = Buffer(secretKey, "utf8");
 secretKey = crypto.createHash("md5").update(secretKey).digest("hex");
 secretKey = new Buffer(secretKey, "hex");
 var cipher = crypto.createDecipheriv("aes-128-cbc", secretKey, iv), coder = [];
 coder.push(cipher.update(data, "hex", "utf8"));
 coder.push(cipher.final("utf8"));
 return coder.join("");
}
 
var iv = new Buffer("1234567890123456","utf8");
var key = "3d8be02c70665eaba6877a67ce3d49b2";
var data = '张冰冰';
var enc_str = cbcEncrypt(data,key, iv);
var dec_str = cbcDecrypt(enc_str,key, iv);
console.log("密文："+ enc_str," / 明文："+dec_str);
 

```

## Java加密解密

```java
public class AesTest2 {
    public static String cbcEncrypt(String content, String secretKey, byte[] iv) throws Exception {
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] thedigest = md.digest(secretKey.getBytes("UTF-8"));
        SecretKeySpec skc = new SecretKeySpec(thedigest, "AES");
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        //算法参数
        AlgorithmParameterSpec paramSpec = new IvParameterSpec(iv);
        cipher.init(Cipher.ENCRYPT_MODE, skc, paramSpec);
        byte[] input = content.getBytes("UTF-8");
        int len = input.length;
        byte[] cipherText = new byte[cipher.getOutputSize(len)];
        int ctLength = cipher.update(input, 0, len, cipherText, 0);
        ctLength += cipher.doFinal(cipherText, ctLength);
        return byte2hex(cipherText);
    }

    /**
     * AES-128-CBC对称解密
     * @param encrypted 解密的密文
     * @param secretKey 密钥
     * @param iv CBC向量，长度为16的byte数组
     * @return 明文
     * @throws Exception
     */  public static String cbcDecrypt(String encrypted, String secretKey, byte[] iv) throws Exception {
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] thedigest = md.digest(secretKey.getBytes("UTF-8"));
        SecretKeySpec skey = new SecretKeySpec(thedigest, "AES");
        Cipher dcipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        //算法参数
        AlgorithmParameterSpec paramSpec = new IvParameterSpec(iv);
        dcipher.init(Cipher.DECRYPT_MODE, skey, paramSpec);
        byte[] clearbyte = dcipher.doFinal(toByte(encrypted));
        return new String(clearbyte);
    }

    /**
     * 将十六进制字符串转换成字节数组
     *
     * @param hex 十六进制字符串
     * @return byte数组
     */
    public static byte[] toByte(String hex) {
        byte[] binary = new byte[hex.length() / 2];
        for (int i = 0, l = binary.length; i < l; i++) {
            binary[i] = (byte) Integer.parseInt(hex.substring(2 * i, 2 * i + 2), 16);
        }
        return binary;
    }

    /**
     * 字节数组转换为二行制表示
     *
     * @param inStr 需要转换字节数组
     * @return 字节数组的二进制表示
     */
    public static String byte2hex(byte[] inStr) {
        StringBuilder out = new StringBuilder(inStr.length * 2);
        for (int i = 0, l = inStr.length; i < l; i++) {
            // 字节做"与"运算，去除高位置字节 11111111
            if (((int) inStr[i] & 0xff) < 0x10) {
                //小于十前面补零
                out.append("0");
            }
            out.append(Long.toString((int) inStr[i] & 0xff, 16));
        }
        return out.toString();
    }

    public static void main(String[] args) {
        try {
            String data = "你好";
            byte[] iv = "1234567890123456".getBytes("UTF-8");
            String key = "3d8be02c70665eaba6877a67ce3d49b2";
            String enc = cbcEncrypt(data, key, iv);
            System.out.println("encoding: " + enc);
            System.out.println("decoding: " + cbcDecrypt(enc, key, iv));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```