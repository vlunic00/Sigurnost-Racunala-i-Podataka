# Izvještaj za 2. labaratorijske vježbe

Created: November 26, 2021 10:47 AM
Last Edited Time: November 26, 2021 11:23 AM

## Zadatak

Pronaći svoje ime u enkriptiranim nazivima datoteka, te dekriptirati tu datoteku i otkriti njen sadržaj.

## Izvedba

Vježba se izvodi u virtualnom python okruženju kako naš rad ne bi komprimitirao globalno okruženje.

Kako bi vježba radila potrebno je instalirati *cryptography* modul za python naredbom `$pip install cryptography`

Iz cryptography modula importamo Fernet metodu koja nam omogućuje simetričnu enkripciju.

Koristimo naredbu `from cryptography.fernet import Fernet`

### Dekripcija imena datoteke

Korištenjem sljedećeg isječka koda, otkrijemo koja datoteka u serveru pripada nama.

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

if __name__ == "__main__":
    h = hash('milicevic_mario')
    print(h)
```

 Znamo koja hash funkcija je korištena u enkripciji imena datoteke, te znamo plaintext ime datoteke, pa je lako otkriti njen ciphertext.

### Dekripcija sadržaja datoteke

Za dekripciju sadržaja datoteke potrebno je izvesti *brute-force* napad.

On je validna opcija jer ključ ima entropiju samo 22 bita.

Pomoću Fernet metode generiramo ključeve kojima ćemo isprobavati dekriptirati datoteku dok ne pronađemo ključ.

S obzirom da znamo da se radi o .png formatu, znat ćemo da smo uspijeli kada otkrijemo png header u sadržaju.

Koristimo sljedeći isječak koda:

```python
def test_png(header):
    if header.startswith(b"\211PNG\r\n\032\n"):
        return True

def brute_force():
    # Reading from a file
    filename = hash('lunic_vedran') + ".encrypted"
    with open(filename, "rb") as file:
        ciphertext = file.read()

    ctr = 0
    while True:
            key_bytes = ctr.to_bytes(32, "big")
            key = base64.urlsafe_b64encode(key_bytes)
            if not (ctr+1) % 1000:
                print(f"[*] Keys tested: {ctr+1:,}", end="\r")

            try:
                plaintext = Fernet(key).decrypt(ciphertext)
                header = plaintext[:32]
                if test_png(header):
                    print(f"[+] KEY FOUND: {key}")
                    # Writing to a file
                    with open("BINGO.png", "wb") as file:
                        file.write(plaintext)
                    break
            except Exception:
                pass
            ctr += 1

if __name__ == "__main__":
    brute_force()
```

Iako ključ ima malu entropiju, napadu je potrebno par desetaka minuta za dekripciju.

Proces možemo ubrzati korištenjem paralelizacije.