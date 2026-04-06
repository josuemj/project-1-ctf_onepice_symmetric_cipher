# Reto 1 - Desafío 1: Monkey D. Luffy (XOR)

1. Levantando el contenedor luffy
```bash
docker compose -p onepiece up -d luffy
```
![contenedor reto 1 levantado](./img/c1/container.png)

2. Entrando al contenedor
```bash
docker compose -p onepiece exec luffy bash
```
![contenedor reto 1 levantado](./img/c1/intoit.png)

3. Navegando a la carpeta del reto (flags y .zip)
```bash
find ONEPIECE -name "flag.txt" | head -5
find ONEPIECE -name "*.zip" | head -5
```

![navegación reto 1](./img/c1/navigation1.png)

4. Encontrando los archivos correctos mediante el marcador del carné
```bash
find . -name ".marker_*"
# Resultado:
# ./East_Blue/Arlong_Park/Casa_de_Arlong/.marker_238
# ./East_Blue/Baratie/Casa_de_Zeff/.marker_238
```

- **flag.txt** → `ONEPIECE/East_Blue/Arlong_Park/Casa_de_Arlong/flag.txt`
- **ZIP del Poneglyph** → `ONEPIECE/East_Blue/Baratie/Casa_de_Zeff/data_94519abff613.zip`

5. Extrayendo el ZIP del Poneglyph (requiere p7zip por cifrado AES)
```bash
sudo apt-get install -y p7zip-full
7z x data_94519abff613.zip -ponepiece
```

6. Obteniendo el texto oculto en los metadatos EXIF de la imagen
```bash
python3 -c "import re; data=open('poneglyph.jpeg','rb').read(); [print(t.decode()) for t in re.findall(b'[ -~]{6,}', data)]"
# Primera línea del output (hex cifrado):
# 605d51505912585c50595756134d5f5712604d45534513715646126350455346564a1b12515f585e5f5b5d5e177e47555f4e1245524a1740574049585c415a5b5b5712555645125a564b17515d5d4d5e5c47565d17574a5a4a43575c505c1b
```

7. Descifrando el texto EXIF con XOR usando el carné
```python
key = '22397'
ct = bytes.fromhex('605d51505912585c50595756134d5f5712604d45534513715646126350455346564a1b12515f585e5f5b5d5e177e47555f4e1245524a1740574049585c415a5b5b5712555645125a564b17515d5d4d5e5c47565d17574a5a4a43575c505c1b')
pt = bytes([ct[i] ^ ord(key[i % len(key)]) for i in range(len(ct))])
print(pt.decode())
# Output: Robin joined the Straw Hat Pirates, claiming Luffy was responsible for her continued existence,
```

8. Descifrando el flag.txt con XOR usando el carné
```bash
cat ~/ONEPIECE/East_Blue/Arlong_Park/Casa_de_Arlong/flag.txt
# Output hex: 747e727e680501055b075456515b07060252005550530b0f005750055d075457510d540351
```

```python
key = '22397'
ct = bytes.fromhex('747e727e680501055b075456515b07060252005550530b0f005750055d075457510d540351')
pt = bytes([ct[i] ^ ord(key[i % len(key)]) for i in range(len(ct))])
print(pt.decode())
# FLAG_736b0fdbb040a9bba867eb6d0feb4c1c
```

**Flag:** `FLAG_736b0fdbb040a9bba867eb6d0feb4c1c`

# Reto 2 - Desafío 2: Roronoa Zoro (RC4)

1. Levantando el contenedor zoro
```bash
docker compose -p onepiece up -d zoro_image
docker start zoro_challenge && docker exec -u zoro -it zoro_challenge bash
# Contraseña del usuario zoro: FLAG_736b0fdbb040a9bba867eb6d0feb4c1c (flag reto 1)
```
![contenedor reto 2 levantado](./img/c2/intoit.png)

2. Encontrando los archivos correctos mediante el marcador del carné
```bash
find ONEPIECE -name ".marker_*"
# Resultado:
# ./Alabasta/Alubarna/Casa_de_Igaram/.marker_238
# ./Alabasta/Katorea/Casa_de_Toto/.marker_238
```

- **flag.txt** → `ONEPIECE/Alabasta/Katorea/Casa_de_Toto/flag.txt`
- **ZIP del Poneglyph** → `ONEPIECE/Alabasta/Alubarna/Casa_de_Igaram/data_e397616f2a84.zip`

3. Extrayendo el ZIP del Poneglyph (contraseña: flag reto 1)
```bash
cd ~/ONEPIECE/Alabasta/Alubarna/Casa_de_Igaram/
sudo apt-get update && sudo apt-get install -y p7zip-full
7z x data_e397616f2a84.zip -pFLAG_736b0fdbb040a9bba867eb6d0feb4c1c
```
![Desempaquetando](./img/c2/zippping.png)

4. Obteniendo el texto oculto en los metadatos EXIF de la imagen
```bash
python3 -c "import re; data=open('poneglyph.jpeg','rb').read(); [print(t.decode()) for t in re.findall(b'[ -~]{6,}', data)]"
# Primera línea del output (hex cifrado):
# 535c5719434053455c5b5756134e5e465a134d5f575f134d5812615840475b575219405a57415c17535c5c4d5f57401369585c5754554e425a134e5641125b5c5b561c136d5f5740561940534113581745535f55175b5c134d5f57126051565c565c4b5e535c134b425b5c401954404b434d5e51535f554e1256564a54405b51505955
```

5. Descifrando el texto EXIF con XOR usando el carné
```python
key = '22397'
ct = bytes.fromhex('535c5719434053455c5b5756134e5e465a134d5f575f134d5812615840475b575219405a57415c17535c5c4d5f57401369585c5754554e425a134e5641125b5c5b561c136d5f5740561940534113581745535f55175b5c134d5f57126051565c565c4b5e535c134b425b5c401954404b434d5e51535f554e1256564a54405b51505955')
pt = bytes([ct[i] ^ ord(key[i % len(key)]) for i in range(len(ct))])
print(pt.decode())
# Output: and traveled with them to Skypiea where another Poneglyph was held. There was a wall in the Shandorian ruins cryptically describing
```

6. Descifrando el flag.txt con RC4 usando el carné
```bash
cat ~/ONEPIECE/Alabasta/Katorea/Casa_de_Toto/flag.txt
# Output hex: 0ce4feaf3b3b7f83dc7a5bd1ca5d8e5d0c699e7e56a6f5cc31c4dbf1d7f66dab10449f8c02
```

```python
# Referencia: utils/zoro_rc4.py
def KSA(key):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S

def PRGA(S, length):
    i = j = 0
    ks = []
    for _ in range(length):
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        ks.append(S[(S[i] + S[j]) % 256])
    return ks

key = [ord(c) for c in '22397']
ct = bytes.fromhex('0ce4feaf3b3b7f83dc7a5bd1ca5d8e5d0c699e7e56a6f5cc31c4dbf1d7f66dab10449f8c02')
S = KSA(key)
ks = PRGA(S, len(ct))
pt = bytes([ct[i] ^ ks[i] for i in range(len(ct))])
print(pt.decode())
# FLAG_fc278347c746eea4012d5bafde5a5aaa
```

**Flag:** `FLAG_fc278347c746eea4012d5bafde5a5aaa`
