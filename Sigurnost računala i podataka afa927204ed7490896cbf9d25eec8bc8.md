# Sigurnost računala i podataka

### **LAB 2** - **Symmetric key cryptography - a crypto challenge**

Cilj druge laboratorijske vježbe bio je riješiti odgovarajući crypto izazov, odnosno dešifrirati ciphertext u kontekstu simetrične kriptografije.

Na početku vježbe kreirali smo python virtualno okruzenje (omogućava izbjegavanje kolizija između različitih verzija libraryija) pod nazivom "env".

```jsx
python -m venv env
```

Idućim smo se komandama pozicionirali u odgovarajući direktorij i pokrenuli skriptu.

```jsx
cd env ; cd .\Scripts\ ; .\activate
```

Za enkripciju je korištena Python biblioteke *cryptography* pa smo istu i instalirali.

```jsx
pip install cryptography
```

Potom otvaramo python interaktivni shell naredbom "python".

*Plaintext* koji trebamo otkriti enkriptiran je korištenjem *high-level* sustava za simetričnu enkripciju iz navedene biblioteke - [**Fernet**](https://cryptography.io/en/latest/fernet/).

**`from** **cryptography.fernet** **import** Fernet`

Upoznali smo se s osnovnim funkcionalnostima Ferneta:

**Generiranje ključa:**

`key = Fernet.generate_key()`

**Inicijalizacija fernet sustava pomoću dobivenog ključa:**

`f = Fernet(key)`

**Enkripcija proizvoljne varijable *plaintext:***

`ciphertext = f.encrypt(plaintext)`

**Dekripcija varijable ciphertext:**

`deciphertext = f.decrypt(ciphertext)`

## Crypto challenge

Pristupamo serveru [http://a507-server.local](http://a507-server.local/) na kojem se nalaze imena file-ova(imena studenata) dobivena enkriptiranjem koristeći SHA-256 algoritam.

Pozicioniramo se u vlastiti direktorij i kreiramo skriptu.

```jsx
code brute_force.p
```

U skriptu zalijepimo idući kod:

```python
from cryptography.hazmat.primitives import hashes
def hash(input): 
	if not isinstance(input, bytes): 
		input = input.encode() 
	digest = hashes.Hash(hashes.SHA256()) 
	digest.update(input) 
	hash = digest.finalize() 

	return hash.hex()
filename = hash('prezime_ime') + ".encrypted"
```

Vratimo se u CMD i naredbom 

```python
python brute_force.py
```

uz pomoć funkcije

```python
if __name__ == "__main__":
	hash_value = hash("tomic_klara_bruna")
	print(hash_value)
```

izgenerira nam se naše hashirano ime.

Zatim, preuzmemo našu datoteku i spremimo u folder u koji nam je spremljen brute_force.py.

Za enkripciju smo koristili **ključeve ograničene entropije - 22 bita**. Ključevi su generirani uz pomoć Ferneta. Znamo plaintext i ciphertext, trebamo dekriptirati naš file, odnosno saznati key. Koristit ćemo brute-force pristup.

Iteriramo kroz ključeve.

```python
if not(ctr + 1) % 1000:
	print(f"[*] Keys tested: {ctr +1:,}", end="\r"
```

Ispisujemo koje smo ključeve isprobali, odnosno svako tisućiti counter(ključ).

Importamo base64 na vrhu skripte, pozovemo funkciju i onda u CMD pokrenemo skriptu i pratimo koliki se broj ključeva isprobao.

Da bismo uspjeli dekriptirati file trebamo uzeti ciphertext iz file-a i ubacit ga u funkciju pa ćemo plaintext dobiti dekripcijom.  

> plaintext = decrypt(key, ciphertext)
> 

Trebamo saznati je li plaintext koji smo dobili baš traženi plaintext. Zato nakon dekripcije stavljamo if uvjet u kojem provjeravamo je li slika PNG formata. Ako je istinit if uvjet pohranit ćemo pronađeni plaintext i isprintati key koji smo pronašli.

```python
import base64
from cryptography.hazmat.primitives import hashes
from cryptography.fernet import Fernet

def hash(input):
	if not isinstance(input, bytes):
		input = input.encode()
	digest = hashes.Hash(hashes.SHA256())
	digest.update(input)
	hash = digest.finalize()
	return hash.hex()

def test_png(header):
	if header.startswith(b"\211PNG\r\n\032\n"):
		return True

def brute_force():
# Reading from a file
	filename = "9319a3b572e2f5ef8889ac16cad0b2921dacfc62cf84a67cc0df718464911cc0.encrypted"
	with open(filename, "rb") as file:
		ciphertext = file.read()
# Now do something with the ciphertext
	ctr = 0
	while True:
		key_bytes = ctr.to_bytes(32, "big")
		key = base64.urlsafe_b64encode(key_bytes)
		if not (ctr + 1) % 1000:
			print(f"[*] Keys tested: {ctr +1:,}", end = "\r")
		try:
			plaintext = Fernet(key).decrypt(ciphertext)
			header = plaintext[:32]
			if test_png(header):
				print(f"[+] KEY FOUND: {key}")
				# Writing to a file
				with open("BINGO.png", "wb") as file:
					file.write(plaintext)
						break
		except Exception:
			pass
# Now initialize the Fernet system with the given key
# and try to decrypt your challenge.
# Think, how do you know that the key tested is the correct key
# (i.e., how do you break out of this infinite loop)?
		ctr += 1
if __**name__** == "__**main__**":
	brute_force()
```