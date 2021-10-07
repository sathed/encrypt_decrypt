# Encrypting and Decrypting Files
In this repository, are the tools for decrypting and encrypting files using SSL cert. This is simply a demonstration of how encryption works.

## Salt
In these example, you will be 'salting' your encryption. A salt is a randomly generated value that is included in the hashed value. The purpose of a salt is to deter the use of a [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table) in a hack.

#### Salt Example
An example of a username, password, and corresponding hash of the password.

| Username | Password | SHA256 Hash |
| -------- | -------- | ----------- |
| admin    | password1234 | 427bce94ca938c94b4a6cc3cacfca7b8bf294eaa0eb7bc2e777c693df1688aa1 |

An example of a username, password, salt, and corresponding hash of the salt + password (**salt**password).

| Username | Salt | Password | SHA256 Hash |
| -------- | ---- | -------- | ----------- |
| admin    | fd2b46eef359 | password1234 | 176277e42364e34d72dec05a20f846524a264c14266d064e5ed39c96108bd5e2 |
| user1    | cu498e537der | password1234 | b76773467c12491069877ec3b16671fd9fa6d7c81decb2b2d98e0043b152fcb3 |

### Generating SSL Keys
If you already have existing SSL cert, you can use it. If you need to generate one, you can use the following commands. 
```
openssl genrsa -aes256 -passout pass:gsahdg -out server.pass.key 4096
openssl rsa -passin pass:gsahdg -in server.pass.key -out server.key
rm server.pass.key
```
You may also use the keys included in this repository (located in the `ssl` directory), although they will expire eventually.

### Encrypting

1. Create a public key file from your key
   ```
   openssl rsa -in server.key -out publickey.pem -outform PEM -pubout
   ```
   * If you get an error, you probably need to generate a key. See the Generating SSL Keys section above.
2. Create a 10 MB file filled with random data:
   ```
   head -c 10485760 </dev/urandom >message
   ```
3. Generate a random key:
   ```
   openssl rand -hex 64 > bin.key
   ```
4. Encrypt the file using the randomly generated key:
   ```
   openssl enc -aes-256-cbc -salt -in message -out message.enc -pass file:./bin.key
   ```
5. Inspect the two files:
   ```
   file message
   file message.enc
   ```
   You should see:
   ```
   message: data
   message.enc: openssl enc'd data with salted password
   ```
6. Encrypt the random key with the public key:
   ```
   openssl rsautl -encrypt -inkey publickey.pem -pubin -in bin.key -out bin.key.enc
   ```
7. Inspect the new file:
   ```
   file bin.key.enc
   ```
   You should see:
   ```
   bin.key.enc: data
   ```
   This means that you've successfully encrypted the `bin.key` text file.

### Decrypting
Now it's time to decrypt the message. However, this part you will need to figure out on your own. You may need to use some Google-fu. But please try to avoid Google. You can get more information on using the `openssl` tool with the following commands:

```
openssl -h
```
or
```
man openssl
```
**Hint**: Save your decrypted file as something other than `bin.key`. Then, you can compare the hashes of both files like so:

Mac
```
shasum -a 256 /path/to/file
shasum -a 256 /path/to/decrypted_file
```

Linux
```
sha256sum /path/to/file
sha256sum /path/to/decrypted_file
```

Let me know if you get stuck!
