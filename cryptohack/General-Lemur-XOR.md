# Lemur XOR 

## Challenge Overview

The challenge provides **two PNG images** — `lemur.png` and `flag.png` — both encrypted by XOR with the same secret key. The goal is to recover the hidden flag image by XOR-ing the two images together pixel by pixel.

---

## Concept — XOR Cancellation

Since both images were encrypted with the same key:

```
lemur ^ key = lemur_encrypted
flag  ^ key = flag_encrypted
```

XOR-ing the two encrypted images together cancels the key entirely:

```
lemur_encrypted ^ flag_encrypted
= (lemur ^ key) ^ (flag ^ key)
= lemur ^ flag ^ (key ^ key)
= lemur ^ flag
```

This reveals the visual difference between the two original images — which contains the flag.

This works because of XOR's core properties:

```
A ^ B = C  →  A ^ C = B  →  B ^ C = A
```

---

## Exploitation

### Step 1 — XOR the Images Pixel by Pixel

We XOR each RGB channel of every pixel independently:

```python
from PIL import Image

img1 = Image.open("lemur.png").convert("RGB")
img2 = Image.open("flag.png").convert("RGB")

assert img1.size == img2.size

out = Image.new("RGB", img1.size)
pixels1 = img1.load()
pixels2 = img2.load()
pixels_out = out.load()

for x in range(img1.width):
    for y in range(img1.height):
        r1, g1, b1 = pixels1[x, y]
        r2, g2, b2 = pixels2[x, y]
        pixels_out[x, y] = (
            r1 ^ r2,
            g1 ^ g2,
            b1 ^ b2
        )

out.save("result.png")
```

### Step 2 — Read the Output

Opening `result.png` reveals the flag rendered visually in the image.

🏁 Flag retrieved.

---

## Key Takeaway

> **Never encrypt two different messages with the same XOR key.** XOR-ing the two ciphertexts together completely eliminates the key, exposing the relationship between the plaintexts. In the case of images, this leaks the visual content directly. This is the core weakness of key reuse in stream ciphers and one-time pads.
```
