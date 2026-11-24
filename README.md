# vinny-homelab
cybersecurity lab
# SSH Brute-Force Attack with Hydra

## Goal
Simulate a password brute-force attack against my own lab server's SSH service, 
then observe and defend against it.

## Lab setup
- **Attacker:** Kali Linux (192.168.56.103)
- **Target:** Ubuntu Server running OpenSSH (192.168.56.101)
- **Network:** Isolated VirtualBox Host-only network

## Steps

### 1. Confirmed SSH was running on the target
sudo systemctl status ssh

### 2. Created a password wordlist (on Kali)
Included a few common guesses plus the real password, to confirm Hydra could 
succeed.

### 3. Ran Hydra against the target
hydra -l ceezar -P passwords.txt -t 4 ssh://192.168.56.101

Hydra successfully found the correct password after trying the list.

### 4. Viewed the attack from the defender's side
sudo tail -f /var/log/auth.log

Observed multiple `Failed password` entries from the same IP address 
(192.168.56.103) within seconds of each other, followed by a successful login.

### 5. Installed fail2ban as a defense
sudo apt install fail2ban -y
sudo systemctl enable fail2ban --now

### 6. Re-ran Hydra with only wrong passwords
After a few failed attempts, fail2ban banned the attacking IP address 
automatically, confirmed with:
sudo fail2ban-client status sshd

## What I learned
- A brute-force attack is visible in logs as repeated failed logins from one 
  IP in a short time window — this is a key pattern defenders look for.
- Weak or guessable passwords can be found quickly even with a small wordlist.
- fail2ban is an effective first line of defense against this kind of attack, 
  automatically blocking an IP after repeated failures.
- Always test this kind of exercise only in an isolated lab, never against 
  real or production systems.
