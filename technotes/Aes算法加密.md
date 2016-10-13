##app 中网络传输使用Aes加密

主要是核心代码使用```java```编写。

- 什么是AES？

高级加密标准（英语：Advanced Encryption Standard，缩写：AES），是一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。

- AES密钥有什么用？

app与API的网络请求中， API均支持对接口的请求内容和响应内容进行AES加密。加密后，在网络上传输的接口报文内容将会由明文内容变为密文内容，可以大大提升接口内容传输的安全性。

- ####详细方案：

   1，**post请求的Aes解密**
   post请求中body的postparam使用aes加密，建立网络请求后，api服务接受到请求后，**进行解密操作**，然后进行逻辑处理，返回给app。**app对返回结果进行解密**，数据处理。

   2，**get请求的Aes解密**
 get请求中postparam使用aes加密，**使用url+Aes加密后的串**建立网络请求后，api服务接受到请求后，**进行解密操作**，然后进行逻辑处理，返回给app。**app对返回结果进行解密**，数据处理。
 
   3，**key的管理与使用**  
   aes 加密key按照app版本和系统平台（android和ios）随机生成。app与api各自保存，app对应版本只保留一个对应key，api需要建一个aes key的配置文件，用来保存配置。

- Aes加密核心代码：
``` java
public static String encrypt() throws Exception {
  
        String key = "开发者自己的AES秘钥";
        String content = "需要加密的参数";
        String charset = "项目使用的字符编码集";
        String fullAlg = "AES/CBC/PKCS5Padding";
  
        Cipher cipher = Cipher.getInstance(fullAlg);
        IvParameterSpec iv = new IvParameterSpec(initIv(fullAlg));
        cipher.init(Cipher.ENCRYPT_MODE,
                new SecretKeySpec(Base64.decodeBase64(key.getBytes()), "AES"),
                iv);
  
        byte[] encryptBytes = cipher.doFinal(content.getBytes(charset));
        return new String(Base64.encodeBase64(encryptBytes));}
        /**
     * 初始向量的方法, 全部为0. 这里的写法适合于其它算法,针对AES算法的话,IV值一定是128位的(16字节).
     *
     * @param fullAlg
     * @return
     * @throws GeneralSecurityException
     */
    private static byte[] initIv(String fullAlg) throws GeneralSecurityException {
        Cipher cipher = Cipher.getInstance(fullAlg);
        int blockSize = cipher.getBlockSize();
        byte[] iv = new byte[blockSize];
        for (int i = 0; i < blockSize; ++i) {
            iv[i] = 0;
        }
        return iv;
    }
```

- Aes解密核心代码：
``` java
/**
     * 
     * @param content 密文
     * @param key aes密钥
     * @param charset 字符集
     * @return 原文
     * @throws EncryptException
     */
    public String decrypt(String content, String key, String charset) throws Exception {
          
        //反序列化AES密钥
        SecretKeySpec keySpec = new SecretKeySpec(Base64.decodeBase64(key.getBytes()), "AES");
          
        //128bit全零的IV向量
        byte[] iv = new byte[16];
        for (int i = 0; i < iv.length; i++) {
            iv[i] = 0;
        }
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);
          
        //初始化加密器并加密
        Cipher deCipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        deCipher.init(Cipher.DECRYPT_MODE, keySpec, ivParameterSpec);
        byte[] encryptedBytes = Base64.decodeBase64(content.getBytes());
        byte[] bytes = deCipher.doFinal(encryptedBytes);
        return new String(bytes);
          
    }
```

-  Aes key生成代码
``` java
 /** * 生成密钥 * 自动生成base64 编码后的AES128位密钥 * @throws NoSuchAlgorithmException * @throws UnsupportedEncodingException */  
    public static String getAESKey() throws Exception {  
        KeyGenerator kg = KeyGenerator.getInstance("AES");  
        kg.init(128);//要生成多少位，只需要修改这里即可128, 192或256 
        SecretKey sk = kg.generateKey();  
        byte[] b = sk.getEncoded();
        return parseByte2HexStr(b);
}  

```
