from PIL import Image, ImageDraw, ImageFont
import math, random, os, shutil, zipfile
from pathlib import Path

out = Path("/mnt/data/tech_3d_readme")
assets = out / "assets"
assets.mkdir(parents=True, exist_ok=True)

W, H = 1200, 360
frames = []
random.seed(7)

# 3D cube vertices
verts = [
    (-1,-1,-1),(1,-1,-1),(1,1,-1),(-1,1,-1),
    (-1,-1,1),(1,-1,1),(1,1,1),(-1,1,1)
]
edges = [(0,1),(1,2),(2,3),(3,0),(4,5),(5,6),(6,7),(7,4),(0,4),(1,5),(2,6),(3,7)]

# neural nodes around the cube
nodes3d = [
    (-1.45, 0.2, 0.2), (-0.8, 1.35, 0.5), (0.0, 1.55, -0.2),
    (0.9, 1.25, 0.35), (1.45, 0.15, -0.1), (0.9, -1.25, 0.25),
    (0.0, -1.5, -0.3), (-0.95, -1.2, 0.15), (-1.5, -0.25, -0.4)
]
node_links = [(0,1),(1,2),(2,3),(3,4),(4,5),(5,6),(6,7),(7,8),(8,0),(1,6),(3,7)]

def project(p, ang_y, ang_x, scale=95, cx=600, cy=180):
    x,y,z = p
    cy1, sy1 = math.cos(ang_y), math.sin(ang_y)
    x,z = x*cy1-z*sy1, x*sy1+z*cy1
    cx1, sx1 = math.cos(ang_x), math.sin(ang_x)
    y,z = y*cx1-z*sx1, y*sx1+z*cx1
    depth = 1/(1 + 0.18*z)
    return cx + x*scale*depth, cy + y*scale*depth, z

# subtle monospaced font
try:
    font = ImageFont.truetype("DejaVuSansMono.ttf", 14)
except:
    font = None

code_strings = ["01", "AI", "ML", "GO", "PY", "SQL", "API", "CV", "++", "10"]
code_particles = [(random.randrange(20,W-20), random.randrange(-H,H), random.choice(code_strings), random.uniform(.4,1.2)) for _ in range(75)]

for f in range(60):
    im = Image.new("RGB", (W,H), (7,10,14))
    d = ImageDraw.Draw(im)
    ang_y = f * math.pi * 2 / 60
    ang_x = 0.22 + math.sin(f/18)*0.08

    # circuit-like background
    for _ in range(18):
        x = random.randrange(40,W-40)
        y = random.randrange(35,H-35)
        length = random.randrange(30,110)
        d.line((x,y,x+length,y), fill=(14,34,34), width=1)
        d.line((x+length,y,x+length,y+random.choice([-12,12])), fill=(14,34,34), width=1)
        d.ellipse((x-2,y-2,x+2,y+2), fill=(31,83,70))

    # moving code particles
    for x, y0, txt, speed in code_particles:
        y = (y0 + f*speed*5) % (H+40) - 20
        alpha = 1 - abs(y-H/2)/(H/2)
        if alpha > .15:
            d.text((x,y), txt, fill=(22,75,60), font=font)

    # outer orbit ellipses
    for rx, ry in [(245,85),(205,145)]:
        box=(600-rx,180-ry,600+rx,180+ry)
        d.ellipse(box, outline=(19,55,50), width=1)

    # cube
    p2 = [project(v,ang_y,ang_x) for v in verts]
    for a,b in edges:
        x1,y1,z1=p2[a]; x2,y2,z2=p2[b]
        bright = 65 if (z1+z2)/2 > 0 else 35
        d.line((x1,y1,x2,y2), fill=(40, min(180, bright+55), 110), width=2)

    # cube nodes
    for x,y,z in p2:
        r=4 if z>0 else 3
        d.ellipse((x-r,y-r,x+r,y+r), fill=(104,255,132))

    # neural network orbit
    np = [project(n, ang_y*0.72, ang_x*0.7, scale=92) for n in nodes3d]
    for a,b in node_links:
        x1,y1,z1=np[a]; x2,y2,z2=np[b]
        d.line((x1,y1,x2,y2), fill=(25,91,78), width=1)
    for i,(x,y,z) in enumerate(np):
        r=6 if z>0 else 4
        d.ellipse((x-r,y-r,x+r,y+r), fill=(88,235,126))
        # tiny signal pulse
        pulse=(f*3+i*17)%40
        if pulse<20:
            rr=7+pulse//3
            d.ellipse((x-rr,y-rr,x+rr,y+rr), outline=(35,110,82), width=1)

    # central core
    d.ellipse((592,172,608,188), fill=(150,255,170))
    d.ellipse((596,176,604,184), fill=(220,255,225))

    # minimal labels
    d.text((38,32), "NEURAL // COMPUTE", fill=(83,145,116), font=font)
    d.text((38,H-42), "AI  ·  ML  ·  BACKEND  ·  SOFTWARE", fill=(57,102,89), font=font)
    d.text((W-178,32), "SYSTEM ONLINE", fill=(76,150,105), font=font)

    frames.append(im)

gif_path = assets / "tech-neural-3d.gif"
png_path = assets / "tech-neural-3d.png"
frames[0].save(gif_path, save_all=True, append_images=frames[1:], duration=75, loop=0, optimize=False)
frames[0].save(png_path)

readme = r'''<div align="center">

# Karri Sai Krishna Naga Satish Reddy

### `AI/ML Engineer` · `Software Engineer`

<img src="./assets/tech-neural-3d.gif" width="100%" alt="Animated 3D neural network and computing system">

<br>

**I build ML systems, backend services, and software products.**

`Machine Learning` · `Computer Vision` · `Backend` · `Full Stack`

<br><br>

[🌐 Portfolio](https://satishreddykarri.github.io/Portfolio/) ·
[💼 LinkedIn](https://www.linkedin.com/in/karri-sai-krishna-naga-satish-reddy-5ab5b1264/) ·
[🐙 GitHub](https://github.com/satishreddykarri) ·
[📊 Kaggle](https://kaggle.com/karrinagasatishreddy) ·
[⚡ LeetCode](https://leetcode.com/u/satishreddykarri_1342/) ·
[✉️ Email](mailto:satishreddykarri121@gmail.com)

</div>

---

### `SKILLS`

**Languages**  
`Python` `Go` `Java` `JavaScript` `SQL` `Dart`

**AI / ML**  
`Machine Learning` `Computer Vision` `YOLOv8` `OpenCV` `Scikit-learn` `NumPy` `Pandas`

**Software**  
`REST APIs` `Node.js` `Express.js` `React` `Flutter`

**Data & Tools**  
`MongoDB` `Firebase` `Supabase` `Tableau` `Git` `GitHub Actions` `Postman` `DBeaver`

---

<div align="center">

`BUILD · LEARN · SHIP`

<sub>AI/ML · Backend · Software Engineering</sub>

</div>
'''

(out/"README.md").write_text(readme, encoding="utf-8")
zip_path = Path("/mnt/data/tech-github-readme.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for p in out.rglob("*"):
        z.write(p, p.relative_to(out))

print(f"Created: {gif_path}")
print(f"Created: {png_path}")
print(f"Created: {out/'README.md'}")
print(f"Created: {zip_path}")
