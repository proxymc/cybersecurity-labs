# Packet Tracer - File and Data Integrity Checks 🛡  

## 🎯 Objectives  
1️⃣ Recover files after a cyber attack  
2️⃣ Verify file integrity using hashing  
3️⃣ Escalate and transfer compromised files  

## 🛠 Tools Used  
- **Cisco Packet Tracer**  
- **MD5 & HMAC hashing**  
- **FTP for file transfer**  

## 📝 Steps  
### 1️⃣ Recover files from a backup server  
- Connect to the **HQ FTP Server** and retrieve missing files.  
- Restore hashes from the last valid backup.  

### 2️⃣ Verify file integrity using hashing  
- Compute **MD5 hashes** for downloaded files.  
- Compare them to the stored hashes.  

### 3️⃣ Identify altered files & escalate  
- If any hashes don’t match, **report the issue**.  
- Send the compromised file to security analysts.  

### 4️⃣ Use HMAC for added security  
- Compute an **HMAC hash** to verify authenticity.  
- Understand why HMAC is more secure than simple hashing.  

## 💡 Lesson Learned  
Hashing ensures **data integrity**, but **HMAC adds an extra layer of security** by requiring a secret key.  

🔗 **Repository:** [Cybersecurity Labs](../../README.md)  
