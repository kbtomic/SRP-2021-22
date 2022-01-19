# Sigurnost računala i podataka (Lab 6)

## **Lab 6 - Linux permissions and ACLs**

U okviru ove vježbe upoznat ćemo se s postupkom upravljanja korisničkim računima na Linux-u.

**A. Kreiranje novog korisničkog računa**

Otvaramo shell i izvršimo wsl naredbu.

Svaki korisnik ima svoj UID i mora pripadati bar jednoj grupi. U to se možemo uvjeriti naredbama *id* i *groups*.

Dodajemo novi korisnički račun:

```jsx
sudo adduser alice5 
```

Postavljamo lozinku alice.

Logiramo se kao alice i saznajemo odgovarajuće identifikatore korisnika i grupa kojima alice pripada.

```jsx
su - alice5
```

Dodajemo još jedan korisnički račun:

```jsx
sudo adduser bob5 
```

Postavljamo lozinku bob.

Naredbom exit vraćamo se u shell korisnika koji ima administratorske ovlasti.

**B. Standardna prava pristupa datotekama**

Potom se logiramo kao alice i kreiramo novi direktorij *srp* i u njemu datoteku *security.txt*.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled.png)

Izlistajemo informacije o novom direktoriju i datoteci i određujemo vlasnike ovih resursa kao i dopuštenja definirana na njima. Koristimo sljedeće naredbe: `ls -l` ili `getfacl`.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%201.png)

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%202.png)

Onemogućavamo korisniku da čita (read) sadržaj datoteke *security.txt*.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%203.png)

Vraćamo korisniku read dopuštenje nad datotekom *security.txt*.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%204.png)

Ako se logiramo kao Bob možemo pročitati sadržaj datoteke *security.txt*. 

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%205.png)

Preko Alice mičemo prava čitanja s other korisnika (i Boba). 

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%206.png)

Dodajemo boba u grupu alice5 kako bi on kao član grupe mogao pročitati sadržaj datoteke *security.txt*. Prije toga preko exit komande vraćamo se u korisnika s administratorskim ovlastima.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%207.png)

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%208.png)

Potom pokušavamo pročitati sadržaj datoteke */etc/shadow.* Nismo u grupi shadow, nemamo owner prava (nismo root) i other nema nikakva prava pa ne mozemo vidjeti shadow.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%209.png)

Mičemo boba iz grupe alice5.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%2010.png)

**C. Kontrola pristupa korištenjem *Access Control Lists (ACL)***

Bob više nema pristup sadržaju datoteke *security.txt*. 

Želimo Boba dodati u ACL kako bi mogao čitati sadržaj datoteke *security.txt*.

Modificiramo ACL listu.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%2011.png)

Kao što smo ubacili Boba u ACL listu, možemo ubaciti cijelu grupu u ACL.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%2012.png)

Omogućavamo grupi pravo pristupa datoteci *security.txt*.

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%2013.png)

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%2014.png)

**D. Linux procesi i kontrola pristupa**

Svaki linux proces u izvršavanju ima svoj jedinstveni identifikator, *process identifier* PID. Osim toga, svakom od procesa se pridijeli id trenutno logiranog *user-a*, UID. Na temelju UID-ja Kernel će odlučivati ima li proces pristup određenim resursima ili ne.

Trenutno aktivne procese možemo izlistati korištenjem naredbe `ps -ef`.

Oduzimamo Bobu prava čitanja datoteke *security.txt* tako da ga maknemo iz grupe koja ima prava čitanja.

```jsx
gpasswd -d bob alice_reading_group5
```

Otvaramo WSL shell i u direktoriju kreiramo Python skriptu sljedećeg sadržaja: 

```python
import os

print('Real (R), effective (E) and saved (S) UIDs:')
print(os.getresuid())

with open('/home/alice/srp/security.txt', 'r') as f:
print(f.read())
```

Izvršavanjem ove skripte dobili smo *permission denied* jer trenutno logirani user nema nikakva prava nad datotekom.

Ako pokrenemo skriptu kao Bob onda je pokretanje uspješno zbog Bobovih prava.

Ako kao Bob pokrenemo komandu *passwd*, dobit ćemo mogućnost promijeniti lozinku iako nemamo pristup */etc/shadow* folderu. 

Izvršavamo naredbu koja ispisuje tekuće procese sa njihovim stvarnim i efektivnim vlasnicima:

```jsx
ps -eo pid,ruid,euid,suid,cmd
```

![Untitled](Sigurnost%20rac%CC%8Cunala%20i%20podataka%20(Lab%206)%2046268c2c78be4fe6a7d2ef4cdfeae752/Untitled%2015.png)

RUID odgovara Bobu, a EUID super useru.