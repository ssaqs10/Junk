

encode a heading into:

0 … 2π radians  →  0 … 65535   (i.e., 0 … 2^16−1)


Equivalent to:

heading_normalized = scaled / 65536





scaled = headingRad × 65536 / (2π)

👉 Use 65536, then clamp to 0–65535


---


double headingRad = Math.PI / 2; // example

// Normalize to 0…2π
headingRad = (headingRad % (2 * Math.PI) + (2 * Math.PI)) % (2 * Math.PI);

// Convert to 0…65535
int scaled = (int) Math.floor(
        headingRad * 65536.0 / (2 * Math.PI)
);

// Safety clamp
scaled = scaled & 0xFFFF;


---

🧭 Example — 90° (π/2)

π/2 = 0.25 of circle

scaled = 0.25 × 65536 = 16384
Hex = 0x4000

---
Decode back to radians

int scaled = ... & 0xFFFF;

double headingRad =
        scaled * (2 * Math.PI) / 65536.0;


---

 Why 65536 (not 65535)?

Because this representation treats heading as a wrap-around angle:

0  = 0°
32768 ≈ 180°
65535 ≈ 359.9945°
65536 ≡ 0° again
