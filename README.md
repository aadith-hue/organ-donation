# organ-donation
organ donation website using python
from http.server import BaseHTTPRequestHandler, HTTPServer
import urllib.parse
import sqlite3

# Initialize SQLite database
def init_db():
    with sqlite3.connect('donors.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS donors (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT NOT NULL UNIQUE,
                organ TEXT NOT NULL
            )
        ''')
        conn.commit()

init_db()

class RequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write(b"<html><body><h1>Welcome to the Organ Donation Platform</h1>"
                            b"<a href='/register'>Register as a Donor</a><br>"
                            b"<a href='/donors'>View Donors</a></body></html>")
        elif self.path == '/register':
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write(b"<html><body><h1>Register as an Organ Donor</h1>"
                            b"<form action='/register' method='POST'>"
                            b"Name: <input type='text' name='name'><br>"
                            b"Email: <input type='email' name='email'><br>"
                            b"Organ: <input type='text' name='organ'><br>"
                            b"<input type='submit' value='Register'>"
                            b"</form></body></html>")
        elif self.path == '/donors':
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            donors = self.get_donors()
            donor_rows = ''.join(f"<tr><td>{d[0]}</td><td>{d[1]}</td><td>{d[2]}</td><td>{d[3]}</td></tr>" for d in donors)
            self.wfile.write(f"<html><body><h1>List of Organ Donors</h1>"
                             f"<table border='1'><tr><th>ID</th><th>Name</th><th>Email</th><th>Organ</th></tr>"
                             f"{donor_rows}</table></body></html>".encode())
        else:
            self.send_response(404)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write(b"<html><body><h1>404 Not Found</h1></body></html>")

    def do_POST(self):
        if self.path == '/register':
            content_length = int(self.headers['Content-Length'])
            post_data = self.rfile.read(content_length).decode()
            post_params = urllib.parse.parse_qs(post_data)

            name = post_params.get('name', [''])[0]
            email = post_params.get('email', [''])[0]
            organ = post_params.get('organ', [''])[0]

            if name and email and organ:
                self.save_donor(name, email, organ)

            self.send_response(302)
            self.send_header('Location', '/donors')
            self.end_headers()

    def save_donor(self, name, email, organ):
        with sqlite3.connect('donors.db') as conn:
            cursor = conn.cursor()
            cursor.execute('''
                INSERT INTO donors (name, email, organ)
                VALUES (?, ?, ?)
            ''', (name, email, organ))
            conn.commit()

    def get_donors(self):
        with sqlite3.connect('donors.db') as conn:
            cursor = conn.cursor()
            cursor.execute('SELECT * FROM donors')
            return cursor.fetchall()

def run(server_class=HTTPServer, handler_class=RequestHandler, port=8080):
    server_address = ('', port)
    httpd = server_class(server_address, handler_class)
    print(f'Starting httpd server on port {port}...')
    httpd.serve_forever()

if __name__ == '__main__':
    run()
