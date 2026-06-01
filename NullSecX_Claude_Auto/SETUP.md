# CREATE DIRECTORIES

mkdir -p \
targets \
scripts \
templates \
wordlists \
knowledgebase \
tools \
automation

# CREATE CLAUDE.md FILE

Create a CLAUDE.md file and then ccopy the content from GITHUB

# CREATE scripts/recon.sh
---

#!/bin/bash

DOMAIN=$1

mkdir -p targets/$DOMAIN/recon

subfinder -d $DOMAIN -silent | anew targets/$DOMAIN/recon/subdinfder.txt

cat targets/$DOMAIN/recon/subfinder.txt |\
httpx-toolkit -silent | anew targets/$DOMAIN/recon/live.txt

katana -list targets/$DOMAIN/recon/live.txt -silent |\
anew targets/$DOMAIN/recon/urls.txt

---

# CREATE scripts/js.sh
---

#!/bin/bash

DOMAIN=$1

cat targets/$DOMAIN/recon/urls.txt |\
grep "\.js$" |\
anew targets/$DOMAIN/js/jsfiles.txt

---

#CREATE templates/memory.json
---

{
 "technologies":[],
 "endpoints":[],
 "graphql":[],
 "auth":[],
 "roles":[],
 "wafs":[],
 "findings":[],
 "bypasses:[]
}

---

# CREATE scripts/start_target.sh
---

#!/bin/bash

DOMAIN=$1

mkdir -p targets/$DOMAIN/{recon,findings,js,requests,responses,screenshots,payloads}

cp templates/memory.json targets/$DOMAIN/memory.json

echo "[+] Target Initialized: $DOMAIN"

---

# CREATE scripts/xss.sh
---

#!/bin/bash

DOMAIN=$1

mkdir targets/$DOMAIN/xss

cat targets/$DOMAIN/recon/urls.txt |\
gf xss |\
kxss |\
tee targets/$DOMAIN/xss/potential.txt

cat targets/$DOMAIN/xss/potential.txt |\
dalfox pipe \
--silence \
--skip-bav |
-o targets/$DOMAIN/xss/dalfox.txt

---

# CREATE scripts/sqli.sh
---

#!/bin/bash

DOMAIN=$1

mkdir targets/$DOMAIN/sqli

cat targets/$DOMAIN/recon/urls.txt |\
gf sqli |\
tee targets/$DOMAIN/sqli/parameters.txt

while read url; do
 sqlmap -u "$url"\
 --batch\
 --random-agent\
 --level=1\
 --risk=1\
 --threads=2\
 --smart\
 --output-dir=targets/$DOMAIN/sqli/sqlmap
done < targets/$DOMAIN/sqli/parameters.txt

---

