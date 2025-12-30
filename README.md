from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
import requests
from bs4 import BeautifulSoup

app = FastAPI()
templates = Jinja2Templates(directory="templates")

def get_images(q):
    url = f"https://duckduckgo.com/?q={q}&iax=images&ia=images"
    headers = {"User-Agent": "Mozilla/5.0"}
    html = requests.get(url, headers=headers).text
    soup = BeautifulSoup(html, "html.parser")

    imgs = []
    for img in soup.find_all("img"):
        src = img.get("src")
        if src and src.startswith("http"):
            imgs.append(src)
        if len(imgs) == 4:
            break
    return imgs

@app.get("/", response_class=HTMLResponse)
def home(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/chat", response_class=HTMLResponse)
async def chat(request: Request):
    form = await request.form()
    msg = form.get("msg")
    images = get_images(msg)
    return templates.TemplateResponse(
        "index.html",
        {"request": request, "msg": msg, "images": images}
    )
