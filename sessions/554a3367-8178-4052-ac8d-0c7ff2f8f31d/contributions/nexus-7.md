**Problem:** Need a backend-frontend pair that fetches remote data, filters it, and displays it with performance metrics—zero external dependencies for the frontend.

**Solution:** `pipeline.py` runs a lightweight HTTP server using Python's `http.server` module with CORS headers; `index.html` uses vanilla JS to fetch from localhost:8080, render cards with CSS Grid, and measure end-to-end latency via `performance.now()`.

**Implementation:**

**pipeline.py:**
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import urllib.request
from urllib.parse import urlparse

class PostHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            try:
                with urllib.request.urlopen('https://jsonplaceholder.typicode.com/posts') as r:
                    posts = json.loads(r.read())[:10]
                self.send_response(200)
                self.send_header('Content-type', 'application/json')
                self.send_header('Access-Control-Allow-Origin', '*')
                self.end_headers()
                self.wfile.write(json.dumps(posts).encode())
            except Exception as e:
                self.send_response(500)
                self.end_headers()
                self.wfile.write(json.dumps({'error': str(e)}).encode())
        else:
            self.send_response(404)
            self.end_headers()

    def log_message(self, format, *args):
        pass

if __name__ == '__main__':
    server = HTTPServer(('localhost', 8080), PostHandler)
    print('Server running on http://localhost:8080')
    server.serve_forever()
```

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Post Viewer</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, sans-serif; background: #f5f5f5; padding: 20px; }
        .header { max-width: 1200px; margin: 0 auto 20px; }
        h1 { font-size: 28px; margin-bottom: 10px; }
        .meta { color: #666; font-size: 14px; }
        .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(350px, 1fr)); gap: 16px; max-width: 1200px; margin: 0 auto; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .card h3 { font-size: 16px; margin-bottom: 12px; line-height: 1.4; }
        .card p { color: #555; font-size: 14px; line-height: 1.6; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Posts</h1>
        <div class="meta">Records: <span id="count">—</span> | Fetch time: <span id="time">—</span>ms</div>
    </div>
    <div class="grid" id="grid"></div>

    <script>
        (async () => {
            const start = performance.now();
            try {
                const res = await fetch('http://localhost:8080/');
                const posts = await res.json();
                const time = Math.round(performance.now() - start);
                
                document.getElementById('count').textContent = posts.length;
                document.getElementById('time').textContent = time;
                document.getElementById('grid').innerHTML = posts.map(p => 
                    `<div class="card"><h3>${p.title}</h3><p>${p.body}</p></div>`
                ).join('');
            } catch (e) {
                document.getElementById('grid').textContent = `Error: ${e.message}`;
            }
        })();
    </script>
</body>
</html>
```

Execution: `python3 pipeline.py`, then open `index.html` in browser—posts load asynchronously with performance telemetry.