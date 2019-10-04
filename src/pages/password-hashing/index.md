---
title: A Guide to Password Hashing and Keeping Databases Safe
date: '2019-10-04'
spoiler: You. Must. Do. This.
---

Hashing algorithms are one-way functions. They take any string and turn it into a fixed-length “fingerprint” that is unable to be reversed. This means that if the data is compromised, the hacker<!--more--> cannot get the user’s passwords if they were hashed because at no point were they ever stored on the drive without being in their hashed form.

Websites using hashing typically have this workflow:
1. A user creates an account
2. Their password is hashed and stored on the base
3. When the user attempts to log in, the hash of their entered password is compared to the has stored in the database
4. If the hashes match, the user can access the account. If not, a generic error message is sent back such as “Entered invalid credentials” so hackers can’t trace the error to the username or password specifically.

```
hash("hello") = 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824 hash("hellu") = 3937f988aeb57b6fd75b9c71bf17b9658ec97823bab613df438389b0c896b724 hash("danny") = 668e2b73ac556a2f051304702da290160b29bad3392ddcc72074fefbee80c55a
```

__NOTE:__ Only secure, or cryptographic hash functions, can be used for password hashing (SHA256, SHA512, RipeMD, WHIRLPOOL, etc.)

Sadly, just cryptographically hashing passwords does not ensure safety.

---

# Cracking Hashes

## Brute Force and Dictionary Attacks

The easiest way to decrypt a hash is just to guess the password. The way to do this is to guess the user password, hash the guess, and compare it to the hash of the actual password you are trying to solve. If the two hashes match, the unhashed version of the guess is the right password.

A __brute force attack__ goes through every possible combination of characters given a certain length. Even though they will 100% eventually crack any given password, it is difficult to use this method because of how computationally expensive it is. Some passwords that are even fairly short in length can take thousands of years (literally) to crack using brute force.

```
Trying aaa : failed 
Trying aab : failed 
Trying aac : failed 
... 
Trying acb : failed 
Trying acc : success!
```

__Dictionary attacks__ use a file containing commonly used words, phrases, or passwords that are likely to be a used password. There are databases you can find that hold the top 100,000 (or however many) most commonly used passwords. The attack hashes these passwords and compares the hash to the password to crack. For cracking the average Joe Shmo, this is sometimes a good method to use and is certainly faster than using a brute force attack.

__Lookup tables__ can improve cracking performance by pre-computing the hashes so when it comes time to guess the password, the program needs not to spend compute time actually hashing the guesses.

In the next section, we will take a look at “salting” which makes these cracking methods impossible to use reliably.

# Salting

The reason lookup tables, dictionary attacks, and brute force attacks can work is because the passwords are hashed the same way each time. We can randomize the hash by prepending or appending a random string called a salt to the passwords BEFORE hashing.

```
hash("hello") = 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824 hash("hello" + "jHjdbJShdiodb") = 6f7f167a978166ee23b32c9531ce5dc23ae8fc26e412045858d938d11470831f
```

The salt doesn’t have to be secret because an attacker doesn’t know what the salt will be and thus cannot make pre-computed tables for it.
The Dos and Don’ts of Salting

Don’t:
* Reuse the same salt for each password hashed
* Use short salts
* Use weird double hashes (ex: hash(hash(hash(‘mypass’)))) in lou of a salt

Do:
* Generate random salts using a _Cryptographically Secure Pseudo-Random Number Generator_ (__CSPRNG__)
* Generate a new random unique salt for EACH password hashed
* Generate LONG salts

# Salting Workflow

__Storing a Password:__
1. Generate super long salt with a CSPRNG
2. Prepend the salt to the user password and hash it
3. Save the salt and the hash in the database

__Checking a Password:__
1. Get the salt and hash from the database
2. Prepend the salt to the submitted password and hash it
3. Compare the hashes. If they are equal, the password is correct

__NOTE:__ Always always always (shall I add more always’??) hash on the server. Sometimes JavaScript isn’t enabled and a hash won’t work on the client-side. Also, no one else can access the server so it is ensured to be hashed (You can also hash on the client-side if you so choose)