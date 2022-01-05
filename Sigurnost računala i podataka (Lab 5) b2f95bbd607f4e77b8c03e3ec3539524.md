# Sigurnost računala i podataka (Lab 5)

## **Online and Offline Password Guessing Attacks**

### **Online Password Guessing**

Otvaramo bash shell i pingamo lab server da vidimo jesmo li na istoj lokalnoj mreži.

```python
ping a507-server.local
```

Upoznajemo se s naredbom *nmap.* Određujemo range ip adresa koristeći:

```python
nmap -v 10.0.15.0/28
```

S adrese [http://a507-server.local/](http://a507-server.local/) na kojoj su se nalazili docker kontejneri moje grupe, pronalazim svoj kontejner. Pomoću ssh spajam se na isti.

```python
ssh tomic_klara_bruna@10.0.15.6
```

Potom treba unijeti lozinku. Nakon nekoliko pokušaja upisivanja pogrešne lozinke, zaključujemo da sustav nema rate limit i da je savršen za online napad.

Znamo da je lozinka sastavljena od 4 do 6 lowercase engleskih slova abecede kojih ima 26, što znači da postoji:

26^4+26^5+26^6 kombinacija

suma od 4 do 6 = 26^4(26^2+26+1) cca = 26^4*26^2 = 26^6 cca = 2^29

Za pokušaj brute force-anja lozinke koristimo alat *hydra*.

```python
hydra -l tomic_klara_bruna -x 4:6:a 10.0.15.9 -V -t 4 ssh
```

[STATUS] 64.00 tries/min, 64 tries in 00:01h, 321254064 to do in 83659:55h, 4 active - 64 pokušaja po minuti - 2^6

2^29/2^6 = 2^23 minuta potrebno za probijanje šifre

365*24*60 cca = 2^8*2^5*2^6 = 2^19 minuta po godini

2^23/2^19 = 2^4 = cca 16 godina = predugo potrebno vrijeme da bi se isplatilo čekati.

Stoga, koristimo predefinirane dictionary-ije. Dohvaćamo ih sa servera.

```python
wget -r -nH -np --reject "index.html*" http://a507-server.local:8080/dictionary/g5/
```

Hydri se proslijeđuje dictionary.

```python
hydra -l tomic_klara_bruna -P dictionary/g5/dictionary_online.txt 10.0.15.9 -V -t 4 ssh
```

Dictionary ima nešto više od 850 lozinki, a prosječno treba proći polovicu lozinki da bi se pronašla prava, iako je u mom slučaju našlo pravu lozinku tek nakon 95% isprobanih lozinki iz dictionary-ija. Kad smo pronašli ispravnu lozinku dobili smo pristup kontejneru i s lokalnog se računala spojili na remote računalo.

### **Offline Password Guessing**

Dobili smo pristup remote računalu i želimo pronaći lozinke nekih drugih korisnika. 

Prvo smo pronašli folder unutar kojeg se nalaze hashirane lozinke.

```python
sudo /etc/shadow
```

Kopiramo hash vrijednost nekog korisnika u lokalni file imena “hash.txt”.

Koristimo naredbu hashcat za napad.

```python
hashcat --force -m 1800 -a 3 hash.txt ?l?l?l?l?l?l --status --status-timer 10
```

Opet zaključujemo da je potrebno previše vremena za brute-force lozinke i koristimo predefinirani dictionary.

Drugi napad:

```python
hashcat --force -m 1800 -a 0 hash.txt dictionary/g5/dictionary_offline.txt --status --status-timer 10
```

Kad hashcat pronađe lozinku možemo se uspješno povezati na remote računalo kao korisnik čiju smo lozinku probili.