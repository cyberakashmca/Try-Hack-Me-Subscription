Bilkul bhai. **Burp Suite ke “attack types” ko ratne ki jagah condition dekhkar choose karna** seekho. Neeche ek practical cheat sheet hai—especially **Web Pentesting / OSCP-style labs** ke liye.

# 🔥 Burp Suite Attack Types — कब कौन सा लगाना है?

### Sabse pehle ye formula yaad rakho

```text
Request pakdo
   ↓
Parameter identify karo
   ↓
Parameter kis type ka hai?
   ↓
Ek value change karke response dekho
   ↓
Multiple values test karni hain?
   ↓
Burp Intruder Attack Type choose karo
```

---

# 1. Sniper ⭐ — Default Choice

### Meaning

**Ek position ko attack karta hai, baaki request same rakhta hai.**

```text
GET /product?id=123&category=shoes&sort=price

Sniper:
id       ← attack
category ← same
sort     ← same
```

### Kab use kare?

Jab tum **ek parameter at a time** test karna chahte ho.

### Conditions

```text
✓ SQLi
✓ XSS
✓ SSTI
✓ Command injection
✓ Path traversal
✓ IDOR testing
✓ Parameter fuzzing
✓ Interesting parameter identify karna
```

### Example

Original:

```http
GET /product?id=123
```

Payloads:

```text
123'
123"
123 OR 1=1
../../../../etc/passwd
```

### Golden Rule

> **Pehle Sniper try karo.**

---

# 2. Battering Ram 🔨 — Same Payload Everywhere

### Meaning

**Ek hi payload ko saare marked positions par simultaneously lagata hai.**

Example:

```http
POST /login

username=admin
password=test
```

Dono positions select ki.

Payload:

```text
' OR 1=1--
```

Result conceptually:

```http
username=' OR 1=1--
password=' OR 1=1--
```

### Kab use kare?

Jab **same value multiple locations par ek saath** test karni ho.

```text
✓ Same token multiple parameters mein
✓ Same value multiple headers mein
✓ Duplicate parameter behavior
✓ Input consistency testing
```

### Yaad rakho

```text
Sniper       = one position at a time
Battering Ram = same payload everywhere
```

---

# 3. Pitchfork 🍴 — One-to-One Pair Testing

Ye thoda important hai.

### Meaning

**Har payload position ke liye alag payload list hoti hai.**

Example:

```http
username=USER
password=PASS
```

Payload list 1:

```text
admin
test
john
```

Payload list 2:

```text
1234
password
qwerty
```

Burp pairs:

```text
admin : 1234
test  : password
john  : qwerty
```

### Kab use kare?

Jab tumhare paas **matching pairs** already hain.

```text
✓ Username + corresponding password
✓ ID + corresponding value
✓ First name + corresponding last name
✓ Parameter A + parameter B matching datasets
```

### Important

Pitchfork **all combinations nahi banata**.

```text
List A       List B

admin        1234
john         password
bob          qwerty
```

Result:

```text
admin → 1234
john  → password
bob   → qwerty
```

---

# 4. Cluster Bomb 💣 — All Combinations

### Meaning

**Multiple payload positions ki combinations banata hai.**

Example:

```http
username=USER
password=PASS
```

Username list:

```text
admin
root
test
```

Password list:

```text
1234
password
admin
```

Cluster Bomb approximately test karega:

```text
admin : 1234
admin : password
admin : admin

root : 1234
root : password
root : admin

test : 1234
test : password
test : admin
```

### Kab use kare?

Jab:

```text
✓ Multiple parameters
✓ Har parameter ki independent wordlist
✓ All combinations test karne hain
```

### ⚠️ Caution

Combinations bahut rapidly increase ho sakti hain.

```text
100 usernames × 100 passwords
= 10,000 requests
```

Isliye authorized lab/assessment mein hi use karo.

---

# 🧠 Four Attack Types — One Table

|Attack Type|Payload Positions|Lists|Main Idea|
|---|---|---|---|
|**Sniper**|One at a time|1|Ek parameter test|
|**Battering Ram**|All simultaneously|1|Same payload everywhere|
|**Pitchfork**|Multiple|Multiple|Pair-by-pair|
|**Cluster Bomb**|Multiple|Multiple|Every combination|

---

# 🚦 Decision Tree — Kaunsa Attack Choose Karna Hai?

Isko yaad kar lo:

```text
                Intruder use karna hai
                         │
                         ↓
               Kitne parameters?
                    /          \
                   /            \
                 ONE           MULTIPLE
                  │                │
                  ↓                ↓
               SNIPER        Same payload?
                              /          \
                            YES           NO
                             │             │
                             ↓             ↓
                       BATTERING       Matching
                         RAM           pairs?
                                      /     \
                                    YES      NO
                                     │        │
                                     ↓        ↓
                                 PITCHFORK  CLUSTER
                                             BOMB
```

---

# 🔥 Real Pentesting Situations

## Situation 1 — SQL Injection

Request:

```http
GET /products?id=10
```

Sirf `id` suspicious hai.

### Attack:

```text
SNIPER
```

Why?

```text
id ← payload
```

---

# Situation 2 — XSS

```http
GET /search?q=test
```

Sirf `q` test karna hai.

```text
SNIPER
```

Payload examples:

```text
<test>
"test
'><test>
```

---

# Situation 3 — Same payload two parameters mein

```http
POST /search

q=test
filter=test
```

Tum specifically dekhna chahte ho ki **same input dono locations mein ek saath** kya effect karta hai.

```text
BATTERING RAM
```

---

# Situation 4 — Username + password matching list

Tumhare paas:

```text
user1 : pass1
user2 : pass2
user3 : pass3
```

Matching pairs preserve karne hain.

```text
PITCHFORK
```

---

# Situation 5 — Username aur password ke all combinations

Tumhare paas:

```text
Users:
admin
root
test

Passwords:
1234
password
qwerty
```

Har user ko har password ke saath test karna hai.

```text
CLUSTER BOMB
```

---

# ⚡ Intruder vs Repeater — Very Important

Beginners ka common confusion:

> "Har attack Intruder se hi karna hai?"

**Nahi.**

### Repeater

Use when:

```text
1 request
↓
1 change
↓
response observe
↓
again change
```

Example:

```http
id=10
```

Change:

```http
id=11
```

then:

```http
id=12
```

### Intruder

Use when:

```text
1 request
↓
many payloads
↓
automated comparison
```

---

# 🧠 Practical Workflow

Burp mein request mili:

```http
GET /product?id=10&category=shoes
```

### Step 1

Repeater mein bhejo.

### Step 2

Parameters manually change karo:

```text
id
category
```

### Step 3

Interesting parameter identify karo.

Example:

```text
id=10
```

suspicious laga.

### Step 4

Intruder → Positions

```text
id=§10§
```

### Step 5

Attack type:

```text
SNIPER
```

### Step 6

Payload list:

```text
1
2
3
10
100
999
'
"
../
```

### Step 7

Response compare karo:

```text
Status
Length
Words
Response body
Error message
```

---

# 📊 Response Difference Kaise Identify Kare?

Intruder results mein sirf **Status 200** dekhna enough nahi hai.

Check:

|Indicator|Kya indicate kar sakta hai|
|---|---|
|Status change|Different application behavior|
|Response length change|Different content/error|
|Word count change|Different response|
|Redirect|Authentication/access behavior|
|Error message|Input processing issue|
|Response time|Possible processing difference|
|Reflected payload|Reflection|
|Different JSON field|Backend logic difference|

Example:

```text
Payload     Status    Length
--------------------------------
test        200       4521
hello       200       4520
'           500       218
"           500       219
```

Yahan `'` aur `"` interesting hain.

**500 alone vulnerability prove nahi karta**, lekin further investigation ke liye strong signal ho sakta hai.

---

# 🎯 Attack Type Selection — Ultra Short Cheat Sheet

```text
ONE parameter
        ↓
     SNIPER
```

```text
SAME payload
multiple places
        ↓
  BATTERING RAM
```

```text
PAYLOAD A + PAYLOAD B
matching pair
        ↓
   PITCHFORK
```

```text
PAYLOAD A × PAYLOAD B
every combination
        ↓
  CLUSTER BOMB
```

---

# 🛑 Ek Important Rule

**Intruder attack type vulnerability decide nahi karta.**

Attack type sirf ye decide karta hai ki:

> **Burp payloads ko request ke marked positions par kis tarah distribute karega.**

Vulnerability depend karegi application ke behavior par.

So:

```text
Attack Type ≠ Vulnerability
```

Instead:

```text
Attack Type
     ↓
Payload placement strategy
     ↓
Requests
     ↓
Responses
     ↓
You analyze behavior
     ↓
Potential vulnerability
```

---

# 🏆 OSCP/Lab ke liye Priority

Agar tum abhi learning phase mein ho, is order mein master karo:

```text
1. Repeater ⭐⭐⭐⭐⭐
2. Sniper ⭐⭐⭐⭐⭐
3. Battering Ram ⭐⭐⭐⭐
4. Pitchfork ⭐⭐⭐
5. Cluster Bomb ⭐⭐⭐
```

**90% basic Intruder understanding ke liye pehle Sniper + Repeater ko strong karo.** Cluster Bomb ko sirf isliye mat use karo kyunki woh "powerful" hai—pehle samjho ki tumhe **one-to-one pairs** chahiye ya **all combinations**.