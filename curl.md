## 1. `application/json`

**Send JSON directly:**

```bash
	curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://example.com/api
```

**Send JSON from a file:**

```bash
curl -X POST -H "Content-Type: application/json" -d @data.json https://example.com/api
```


---

## 2. `application/x-www-form-urlencoded`

**Send form data directly:**

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "param1=value1&param2=value2" https://example.com/api
```

**Send form data from a file:**

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d @data.txt https://example.com/api
```


---

## 3. `multipart/form-data`

**Send form fields and files:**

```bash
curl -X POST -F "field1=value1" -F "file1=@/path/to/file.jpg" https://example.com/upload
```

*Note: `curl -F` automatically sets the correct `Content-Type: multipart/form-data` header.*
[^2_4][^2_3][^2_5]

---

## 4. `application/octet-stream`

**Send binary data from a file:**

```bash
curl -X POST --data-binary "@file.bin" -H "Content-Type: application/octet-stream" https://example.com/upload
```


---

## 5. `text/plain`

**Send plain text directly:**

```bash
curl -X POST -H "Content-Type: text/plain" -d "Hello, world!" https://example.com/api
```

**Send plain text from a file:**

```bash
curl -X POST -H "Content-Type: text/plain" --data-binary @message.txt https://example.com/api
```

## 6. `application/xml`

**Send XML data directly:**

```bash
curl -X POST -H "Content-Type: application/xml" -d '<note><body>Hi</body></note>' https://example.com/api
```

**Send XML from a file:**

```bash
curl -X POST -H "Content-Type: application/xml" -d @note.xml https://example.com/api
```


---

## 7. Custom Headers from File

**Read headers from a file (e.g., `headers.txt`):**

```bash
curl -H "@headers.txt" https://example.com/api
```