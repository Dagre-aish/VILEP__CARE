# Lens: talk to your images

Paste an image, ask a question by voice or text, and a local Qwen vision model (running in Ollama) answers in text and speech.

How the pieces fit together:

- **Ollama** runs the model on your PC. This is the "backend".
- **`vision-voice.html`** is the web page. It sends the image and question to Ollama.
- **Cloudflare Tunnel** (optional) gives your local Ollama a public link, so a page hosted on Vercel can reach it.
- **Vercel** (optional) hosts the page so others can open it.

---

## 1. One-time setup (Windows)

1. Install Ollama from https://ollama.com and open it once.
2. Download the vision model (about 2 GB):
   ```
   ollama pull qwen3-vl:2b
   ```
   Check the exact name any time with `ollama list`.
3. Allow the web page to talk to Ollama:
   ```
   setx OLLAMA_ORIGINS "*"
   ```
4. Close that Command Prompt and open a new one. `setx` only applies to new windows.

---

## 2. Run it locally (no tunnel needed)

1. Quit Ollama if it is already running in the system tray (right-click its icon, then **Quit Ollama**).
2. Open a Command Prompt and start the server:
   ```
   ollama serve
   ```
   Leave this window open.
3. Check it: open http://localhost:11434 in Chrome. It should say **Ollama is running**.
4. Double-click `vision-voice.html` to open it in **Chrome or Edge**.
5. Use it:
   - Press **Ctrl+V** to paste an image (or drop a file).
   - Click **Speak your question** and allow the microphone, or type the question.
   - Click **Analyze**. The first answer can take 10 to 30 seconds while the model loads.

When opened from your PC, the page automatically talks to `http://localhost:11434`.

---

## 3. Share it online with a tunnel

Use this when the page is hosted on Vercel, or when someone else needs to reach your model. Your PC must stay on, awake and online.

1. Keep `ollama serve` running (window 1).
2. Install the tunnel tool once, in a new Command Prompt (window 2):
   ```
   winget install Cloudflare.cloudflared
   ```
   Then close that window.
3. Start the tunnel in a new Command Prompt (window 3):
   ```
   cloudflared tunnel --url http://localhost:11434 --http-host-header="localhost:11434"
   ```
4. It prints a link like `https://something-random.trycloudflare.com`. That is your public Ollama address.
5. Open `vision-voice.html` in a text editor, find the hidden input with `id="host"`, and set its `value` to that link.

Keep windows 1 and 3 open while people use the page.

**The link changes every time you restart the tunnel.** Repeat step 5 and redeploy when that happens.

---

## 4. Host the page on Vercel

1. Make a folder and put the page in it, renamed to **`index.html`**.
2. In that folder, run:
   ```
   npx vercel
   ```
   Log in when asked, then press Enter through the questions (new project, default settings). This needs Node.js from https://nodejs.org.
3. For the final public link:
   ```
   npx vercel --prod
   ```

Alternative: upload `index.html` to a GitHub repository, then import the repository at https://vercel.com and click **Deploy**.

To update the page later (for example a new tunnel link), edit `index.html` and run `npx vercel --prod` again.

---

## 5. Stop everything

Close the tunnel window (or press Ctrl+C in it), then the `ollama serve` window. Once the tunnel is closed, the public link stops working.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Failed to fetch` in the page | Ollama is not running, or was started before `OLLAMA_ORIGINS` was set. Quit Ollama completely, open a new Command Prompt, and run `ollama serve`. |
| `model 'qwen3-vl' not found` | The model name must match `ollama list`, for example `qwen3-vl:2b`. |
| `ollama serve` says the port is already in use | Ollama is already running. Run `taskkill /F /IM ollama.exe` and `taskkill /F /IM "ollama app.exe"`, then `ollama serve`. |
| Still blocked after that | Run `netstat -ano \| findstr 11434`, then `taskkill /F /PID <number at the end of the line>`. |
| Voice input does not work | Use Chrome or Edge, and allow microphone access for the page. |
| Vercel page cannot reach the model | Check that the tunnel is running and that the link in `index.html` is the current one. |
| Slow answers | Normal for a 2B model on a laptop GPU. It handles one request at a time. |

## Security notes

- Anyone with your tunnel link can use your GPU and any model installed in Ollama. Do not share it widely.
- Do not put API keys inside the HTML file. Anyone can read the page source.
- The image analysis runs on your PC. Browser voice recognition in Chrome may use an online service.
