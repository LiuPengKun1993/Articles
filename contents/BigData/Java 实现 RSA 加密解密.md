# Java 实现 RSA 加密解密

RSA 加密在编程中很常见，iOS 安卓要用，Java 也要用，最近因为大数据的项目要用到，就写了一份，这里备份一下，也希望能帮到要用的朋友们。




以下是整体代码：

```
import org.apache.commons.codec.binary.Base64;
import javax.crypto.Cipher;
import java.security.*;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

/**
 * Hello RSA!
 *
 */
public class RSAUtils
{
    private static Map<Integer,String> keyMap=new HashMap<>();
    public static void main( String[] args ) throws Exception {
        //生成公钥和私钥
        getKeyPair();
        //加密字符串
        String password="liupengkun";
        System.out.println("随机生成的公钥为："+keyMap.get(0));
        System.out.println("随机生成的私钥为："+keyMap.get(1));
        String passwordEn=encrypt(password,keyMap.get(0));
        System.out.println(password+"\t加密后的字符串为："+passwordEn);
        String passwordDe=decrypt(passwordEn,keyMap.get(1));
        System.out.println("还原后的字符串为："+passwordDe);
    }
    /**
     * 随机生成密钥对
     * @throws NoSuchAlgorithmException
     */
    public static void getKeyPair() throws Exception {
        //KeyPairGenerator类用于生成公钥和密钥对，基于RSA算法生成对象
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
        //初始化密钥对生成器，密钥大小为96-1024位
        keyPairGen.initialize(1024,new SecureRandom());
        //生成一个密钥对，保存在keyPair中
        KeyPair keyPair = keyPairGen.generateKeyPair();
        PrivateKey privateKey = keyPair.getPrivate();//得到私钥
        PublicKey publicKey = keyPair.getPublic();//得到公钥
        //得到公钥字符串
        String publicKeyString=new String(Base64.encodeBase64(publicKey.getEncoded()));
        //得到私钥字符串
        String privateKeyString=new String(Base64.encodeBase64(privateKey.getEncoded()));
        //将公钥和私钥保存到Map
        keyMap.put(0,publicKeyString);//0表示公钥
        keyMap.put(1,privateKeyString);//1表示私钥
    }
    /**
     * RSA公钥加密
     *
     * @param str
     *            加密字符串
     * @param publicKey
     *            公钥
     * @return 密文
     * @throws Exception
     *             加密过程中的异常信息
     */
    public static String encrypt(String str,String publicKey) throws Exception {
        //base64编码的公钥
        byte[] decoded = Base64.decodeBase64(publicKey);
        RSAPublicKey pubKey= (RSAPublicKey) KeyFactory.getInstance("RSA").generatePublic(new X509EncodedKeySpec(decoded));
        //RAS加密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE,pubKey);
        String outStr=Base64.encodeBase64String(cipher.doFinal(str.getBytes("UTF-8")));
        return outStr;
    }

    /**
     * RSA私钥解密
     *
     * @param str
     *            加密字符串
     * @param privateKey
     *            私钥
     * @return 铭文
     * @throws Exception
     *             解密过程中的异常信息
     */
    public static String decrypt(String str,String privateKey) throws Exception {
        //Base64解码加密后的字符串
        byte[] inputByte = Base64.decodeBase64(str.getBytes("UTF-8"));
        //Base64编码的私钥
        byte[] decoded = Base64.decodeBase64(privateKey);
        PrivateKey priKey = KeyFactory.getInstance("RSA").generatePrivate(new PKCS8EncodedKeySpec(decoded));
        //RSA解密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE,priKey);
        String outStr=new String(cipher.doFinal(inputByte));
        return outStr;

    }
}


```


在程序中，我们首先利用 `getKeyPair()` 函数生成公钥和私钥并将其保存到 Map 集合中。然后，基于产生的公钥对明文进行加密。针对已经已经加密的密文，我们再次使用私钥解密，得到明文。
上述程序的输出结果为：

```
随机生成的公钥为：MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCs+QkIaMuogm99lcRjV1xT+AuTHlgIT65svpG8MrPnfjApnkQAdT++VwubdU9ULwVthc6GLuXORxakN/Odd+/jd8WY2IJMVttkYR1mhAZQkx9AU2frNbq97gS9menvL7pQk2ji42Sl1gcZ7tY5+XXGN7EYS52+JGdgfuxGoWNMQwIDAQAB
随机生成的私钥为：MIICdgIBADANBgkqhkiG9w0BAQEFAASCAmAwggJcAgEAAoGBAKz5CQhoy6iCb32VxGNXXFP4C5MeWAhPrmy+kbwys+d+MCmeRAB1P75XC5t1T1QvBW2FzoYu5c5HFqQ385137+N3xZjYgkxW22RhHWaEBlCTH0BTZ+s1ur3uBL2Z6e8vulCTaOLjZKXWBxnu1jn5dcY3sRhLnb4kZ2B+7EahY0xDAgMBAAECgYAMAD9LMRIoP9yCZxi4F8CVJtmOvIU5FmYwr0wnNZFb11p6GLv5GClzCFEK2SnG2nhS7/yzPJ+/HxmIDUW+wCqJjXtoMwBicELk43pNW0QAHAlSytIm13jPsEhNvauDwBpUGqATISwv9DjEf6O1BxrgR5nKpiIpR/QWYlrg8phXOQJBANPhkA6g2Hlvr93BVnhUeSfrTtbwPE/rEIgmEMb6nvnEyOT2sVqr6mw+rs1Byrw0p6+rAESxrwO5Mp2O5tv8B20CQQDQ/XGncq8dBZ+MW2V6eYT+AtvNX//hKGYAz2qPMMR+W4kXb+YiH6SWnTH2f4JkY43BRxt3MlsDxv0nudwAG+RvAkB7+dC77nub2rER1U3OTMczh2jzNVBlBsr+jx9j/kNFLFLMPliaEFuziJ3pdiS1KS4xCKK4jyszx4qJTJNihr6lAkEAz60bFN/FHhzpaEumcudw/g4PKG4eUzuW6XU0GejHSh1iBPVAhmZVYwoAjUg2ZdX8FrW3mGJkyUMjbCeodD9DZwJAdBgcQC+8tzkzwct45EmAQWFMNHLvq3TKdDxr+QluIER1SKjjae8ob+JyI89S8AFJE8IobraVtJ4wTfAt0ZIIMw==
liupengkun	加密后的字符串为：EUz2Goo0m5mNk0al87dsrcqIuzIzNAEvzTBJOXUOiujubJgqVPmsM880aahLbKrNjl5ArJPtguznJ9PruwSBUnGoPpAMetHp+23RBkXB2DwAA200qejLyKt1PkeTLJjDrQVhohlsF+K48TaGpTSfDMoPli3E6eTdS+dk1Xib4SI=
还原后的字符串为：liupengkun
```