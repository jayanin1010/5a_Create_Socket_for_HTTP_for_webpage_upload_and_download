# 5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
## AIM :
To write a PYTHON program for socket for HTTP for web page upload and download
## Algorithm



1. **Client uploads a file** by sending an HTTP `POST` request with file content to the server using a TCP socket.
2. **Server receives the POST request**, reads the content, and saves it as a file (`mytext.txt`).
3. **Client sends an HTTP `GET` request** to download the file from the server.
4. **Server handles the GET request** and sends back the file content as the HTTP response.
5. **Client extracts the file data** from the response and writes it to a local file.


<BR>
## Program 

## client.py

      import socket
      
      def send_request(host, port, request, file_data=None):
          with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
              s.connect((host, port))
              s.sendall(request.encode())
              if file_data:
                  s.sendall(file_data)
              response = b""
              while True:
                  chunk = s.recv(4096)
                  if not chunk:
                      break
                  response += chunk
          return response
      
      def upload_file(host, port, filename):
          with open(filename, 'rb') as file:
              file_data = file.read()
              content_length = len(file_data)
              request = (
                  f"POST /upload HTTP/1.1\r\n"
                  f"Host: {host}\r\n"
                  f"Content-Length: {content_length}\r\n"
                  f"Content-Type: application/octet-stream\r\n"
                  f"\r\n"
              )
              response = send_request(host, port, request, file_data)
          return response.decode(errors="ignore")
      
      def download_file(host, port, filename):
          request = f"GET /{filename} HTTP/1.1\r\nHost: {host}\r\n\r\n"
          response = send_request(host, port, request)
          header_end = response.find(b'\r\n\r\n')
          file_content = response[header_end + 4:]
          with open(filename, 'wb') as file:
              file.write(file_content)
      
      if __name__ == "__main__":
          host = 'localhost'
          port = 8080
      
          # Upload file
          upload_response = upload_file(host, port, 'mytext.txt')
          print("Upload response:", upload_response)
      
          # Download file
          download_file(host, port, 'mytext.txt')
          print("File downloaded successfully.")


## server.py


        import http.server
        import socketserver
        import os
        
        PORT = 8080
        UPLOAD_DIR = "."
        
        class Handler(http.server.SimpleHTTPRequestHandler):
            def do_POST(self):
                try:
                    length = int(self.headers['Content-Length'])
                    data = self.rfile.read(length)
        
                    # Save file with a fixed name or modify this to parse filename
                    with open("mytext.txt", "wb") as f:
                        f.write(data)
        
                    self.send_response(200)
                    self.end_headers()
                    self.wfile.write(b"File uploaded successfully.")
                except Exception as e:
                    self.send_response(500)
                    self.end_headers()
                    self.wfile.write(f"Error: {str(e)}".encode())
        
        # Make sure we're in the correct directory
        os.chdir(UPLOAD_DIR)
        
        try:
            with socketserver.TCPServer(("", PORT), Handler) as httpd:
                print(f"[*] Server running on http://localhost:{PORT}")
                httpd.serve_forever()
        except Exception as e:
            print(f"[!] Server error: {e}")


## OUTPUT

![image](https://github.com/user-attachments/assets/660f9146-06bf-47a2-ac31-eaee565efaca)


## Result
Thus the socket for HTTP for web page upload and download created and Executed
