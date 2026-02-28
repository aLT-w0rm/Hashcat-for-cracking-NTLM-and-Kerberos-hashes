# Cracking-NTLM-and-Kerberos-Hashes-Homelab
* **OBJECTIVE:** I have been given a set of lab-created NTLM and Kerberos hashes to crack using `hashcat` as my tooling.
  * **SCOPE:** Provided 5 NTLM and 1 Kerberos hash.
  * **TOOLS:** Hashcat, hash-identifier (Kali), RockYou.txt, custom wordlists, OneRuleToRuleThemAll.rule, best66.rule.
  * **FOCUS:** Familiarity with options surrounding hashcat, using rules, masks, and custom wordlists.  Incremental and combinator attacks as well.

---

# STEP 1. Identify the Hashes.

* My first process is to identify the hashes.  I actually originally mistook these for MD5, and only found out later that they were actually NTLM.  I was able to use an online platform (hashes.com) to identify them, but it also quickly ran the first hash through its rainbow table and spoiled the first answer.  I laughed, but I was a bit disappointed I didn't get to do it myself.
  * To remedy this, I actually switched to a different online platform for identification, and then I also used `hash-identifier` on my Kali machine.  Finally, I've concluded that they were NTLM.  I was able to identify the final hash as a Kerberos hash after some initial online searching.  This was my first time witnessing a Kerberos hash.
  * **RESULT:** Platform immediately cracked first hash via rainbow table.
  * **ANALYSIS:** Slightly disappointing. Lesson learned — separate identification from cracking for learning purposes.

* Switched platforms for identification only.
* *Kali*> `hash-identifier`
  * **RESULT:** Confirmed NTLM.
  * **ANALYSIS:** Final determination: NTLM (-m 1000).

* Noticed additional hash at bottom:
  * Begins with `$krb5tgs$23$*`
  * **ANALYSIS:** Identified as Kerberos 5 TGS-REP hash.
    * Mode: `-m 13100`
    * Learned this is typically obtained from Kerberoasting.
    * Learned Kerberos hashes tell you in the front of them what type of hash they are.  The above-mentioned hash is type 23, as denoted by the `23` in the beginning.

---

# STEP 2. Initial NTLM Crack Attempt (Basic Wordlist).

* Time to crack.  Literally my first thought was to just use what I've learned and to follow the scope of the project, which was to use Hashcat solely, and not John the Ripper, or other online rainbow tables.  So, I don't have much experience, but I do have some wordlists.  I look up the NTLM hashes code in hashcat and get to work.
* Originally, I thought I could just try and use a blind mask attack, like a ?a?a?a?a?a?a?a type of attack, however, this is wholly unreasonable and obviously way out of scope.  If the objective was to learn and crack passwords, I'd literally learn nothing from just waiting 5 weeks running a completely blind password attempt without even knowing how to use an incremental attack, and at this point I don't know anything about these passwords (length, hints, organizational structure, password structure, etc.)  So, I skipped this and moved on to my first attempt.
    * *Kali*> ` hashcat -m 1000 <hashes.txt> /usr/share/wordlists/rockyou.txt `  This is about as basic as a command I can run at this point.  Hands on experience from platforms like TryHackMe haven't prepared me for much else as of yet.  
      * **RESULT:** The result was disappointing.  In about 30 seconds, it petered out and I had no matches.  I was a bit sad, but, still determined to move forward.
      * **ANALYSIS:** Basic RockYou alone insufficient.

---

# STEP 3. Wordlist + Rules Attack.

At this point, I decided, "Well, maybe I just need a bigger word list!"  So I started looking through for some bigger word lists.  I'd corresponded with my mentor who'd given me some pointers and some learning material on options in hashcat, like using wordlists + rules, masks, and blind brute force attacks.  Afterwards, I started a OneNote journal to start recording my own notes for each new application I use, which has already helped a lot already as well.  Anyways, I was able to try something new, and I used an optional ruleset with my wordlist.

* *Kali*>  ` hashcat -m 1000 -a 0 -w 3 -O hashes.txt rockyou.txt --rules-file=OneRuleToRuleThemAll.rule --session=RockYou_OneRuleToRuleThemAll `

  * **RESULT:** Cracked all NTLM hashes in under 5 minutes.
  * **ANALYSIS:** Rules are wild and unlock the real potential of applications like Hashcat.

---

# STEP 4. Kerberos (TGS-REP) Attempts.

Finally, my actual challenge was the Kerberos.  I did some learning online as to how to identify what hashcat mode I wanted to use for the kerberos, and it seems that most actually tell you right in the beginning.  The hash begins with; `      $krb5tgs$23$*     ` and the "23" in it tells me that it's a "Kerberos 5 TGS-REP" type hash (-m 13100) and I also learned it's typically acquired from a Kerberoasting attack!  Very interesting stuff!

  * **RESULT:** Understanding of how to indentify Kerberos hashes and that there are different kinds.
  * **ANALYSIS:** Kerberos explains what type it is in the prepended hash.  `$krb5tgs$23$*` → `-m 13100`

## Attempt 1 – Previous Attempts.

* `hashcat -m 13100 hash.txt rockyou.txt`
  * **RESULT:** Nothing.
* RockYou + OneRuleToRuleThemAll
* RockYou + best66
  * **RESULT:** Nothing.
  * **ANALYSIS:** Likely structured password, gonna be a bit more complicated.

---

# STEP 5. Custom Wordlist + Mask Attempts.

I spoke with my mentor again at this time, to report that I was able to get the NTLM passwords, but that I was still having a bit of trouble with the Kerberos!  They supplied me with a small hint, that a lot of Kerberos passwords are often structured like |SEASON|YEAR|SYMBOL| .  So, while I was amost positive all of these words would be in RockYou, I actually made my own super small custom list targeted for this type of attack.

Summer  
summer  
Winter  
winter  
Fall  
fall  
Autumn  
autumn  
Spring  
spring  

Tried mask combinations:
`hashcat -a 3 -m 13100 -O -w 3 hash.txt seasons.txt?d?d?s` - Winter24!
`hashcat -a 3 -m 13100 -O -w 3 hash.txt seasons.txt?d?d?s?s` - Winter24!!
`hashcat -a 3 -m 13100 -O -w 3 hash.txt seasons.txt?d?d?d?d?s` - Winter2024!
`hashcat -a 3 -m 13100 -O -w 3 hash.txt seasons.txt?d?d?d?d?s?s` - Winter2024!!

  * **RESULT:** Nothing.
  * **ANALYSIS:** Something's not right here.

---

# STEP 6. Incremental Mask Attack (With Hint).

I think my mentory has just been playing with me, but, it really did teach me a lot.  Both patience and experience.  Trying new techniques like masks and rules, all things I'd never tried at all.  So, I talked to them once more, and was given the big fatty ultimate hint.  `Winter?a?a?a?a?a?a`  However, I wasn't sure how many characters it was, since this was an incremental attack, meaning that it would try `?a` and then after that's exhausted, it'd try `?a?a`, and so forth.

* *Kali*> ` hashcat -a 3 -m 13100 -O -w 3 --increment hash.txt Winter?a?a?a?a?a?a `

  * **RESULT:** Sound the trumpets.  Took a bit, but we got it!  Success!
  * **PASSWORD:** `Winter1shere`
  * **ANALYSIS:** Structured guessing + incremental brute force is extremely effective. Blind brute force would’ve been pointless.

---

# FINAL RESULTS

## NTLM

697a09ac96489c63db58f34c6147dad6 :: hottub  
71179ab187e4bfde6a36d2a0111c5751 :: October123!  
efcb36a246ffd8cd962124097a91e720 :: P4ssw0rd!!  
18d7ffcb18b9b4df82c9443653afdf6b :: Welcome123456789  
33d67d02c684b1d95e5c08c23e812445 :: Welcome1234!!  

## Kerberos

[kerberos hash] :: Winter1shere  

---

# Observations

* Proper hash identification matters.
* Rules dramatically expand wordlist effectiveness.
* Context-based custom lists are powerful.
* Incremental masks are great when partial structure is known.
* A funny little side note is that if I use `hashcat -m 0 hashes.txt --show` OR `hashcat -m 1000 hashes.txt --show` both yield the same results, but if I try with a different mode, like 13100, it doesn't show them.  Interesting.

---

# PROJECT STATUS

✅ NTLM Hashes Cracked  
✅ Kerberos Hash Cracked  
✅ Learned Rules + Masks Properly  
✅ Documented Process  

-aLT_w0rm
