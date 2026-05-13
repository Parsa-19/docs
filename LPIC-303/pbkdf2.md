## PBKDFs2
Is a simple cryptographic Key Drivation Function which prevents dictionary attacks and rainbow tables.

#### KDF
Key Drivation Function is a method and/or cryptographic algorithm in which strengthen weak passwords by adding for example salt or hasing techniques to prevent bruit-force attacks or any threat.<br>
Key drivation function targets on truning low-entropy inputs like human-readable passwords to high-entropy cryptographic keys.

PBKDFs2 stands on Password-Based-Key-Driven-Functions-2 that its purpose is strengthening hashes with salts and iterations.<br>
It is now a known standard defined in [RFC 2898](https://www.rfc-editor.org/info/rfc2898).<br>
It enhances the security of hashes and passwords in two ways:

- salting: with salt the algorithm everytime adds a random value to the password before hashing it and prevents the hash to be always the same sync the salt everytime changes.

- iteration: will create a loop to generate the hash everytime on the previouse one form the password for many times (thousands or even millions) with the salt combination. this process is called streching in which it will slow the computation of hash resaulting the prevention of brute-force attacks.

Implementation of PBKDFs2 by python:
```
import hashlib
import binascii
from os import urandom

def pbkdf2_hash(password, salt=None, iterations=100000):
    if not salt:
        salt = urandom(16)
    dk = hashlib.pbkdf2_hmac('sha512', password.encode(), salt, iterations)
    return binascii.hexlify(dk).decode(), binascii.hexlify(salt).decode()

password = "SecurePassword123"
hash, salt = pbkdf2_hash(password)
print(f"Hash: {hash}\nSalt: {salt}")
```
