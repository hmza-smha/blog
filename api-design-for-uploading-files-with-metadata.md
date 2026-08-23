[Back](./README.md)

# API design for uploading files with metadata



For file upload APIs with per-file metadata, the dominant pattern is **`multipart/form-data`**, not base64-in-JSON. Base64 inflates payload size ~33%, forces the whole thing into memory on both ends, and can't be streamed to disk — it's only really justified when you're stuck behind a client or gateway that can't do multipart.

Within multipart there are three common variants, roughly in order of how often you see them:

**1. One JSON part + N file parts, linked by part name (most robust)**

```
Content-Type: multipart/form-data; boundary=X

--X
Content-Disposition: form-data; name="metadata"
Content-Type: application/json

{"files":[
  {"ref":"f1","title":"Invoice","category":"finance"},
  {"ref":"f2","title":"Receipt","category":"finance"}
]}
--X
Content-Disposition: form-data; name="f1"; filename="invoice.pdf"
Content-Type: application/pdf
<bytes>
--X
Content-Disposition: form-data; name="f2"; filename="receipt.jpg"
Content-Type: image/jpeg
<bytes>
--X--
```

Each metadata object carries an explicit `ref` matching the part name. This is what Google Drive, Slack, and most mature APIs converge on. It survives reordering, partial failures, and proxies that don't preserve part order.

**2. Positional / index-based**

`files[]` repeated plus `metadata[]` repeated, matched by ordinal. Simpler, and it does work — the multipart spec preserves ordering — but it's fragile in practice: some client libraries and middleware reorder or regroup parts, and a single mismatch silently corrupts every pairing after it. Fine for internal APIs, risky for public ones.

**3. Two-phase / pre-signed URLs (best at scale)**

`POST /uploads` returns pre-signed URLs → client PUTs bytes straight to S3/blob storage → client `POST`s the metadata with returned storage keys. Your API never touches the bytes, so you get resumability, no request-size ceiling, and no timeouts on large files. This is the standard for anything with files over a few MB.

---
[Back](./README.md)
