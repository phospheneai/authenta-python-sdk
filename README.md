# Authenta Python SDK  
A Python SDK for the Authenta API to detect deepfakes and manipulated media.

## Getting started

### 1. Create a Client and Run a Simple Detection
```python
from authenta import AuthentaClient

client = AuthentaClient(
    base_url="https://dev.console.authenta.ai/api",  # or https://platform.authenta.ai/api
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET",
)

# Upload a file and wait for processing to finish
media = client.process("samples/nano_img.png", model_type="AC-1")

print("Status:", media["status"])
print("Fake:", media.get("fake"))
print("Result URL:", media.get("resultURL"))
print("Heatmap URL:", media.get("heatmapURL"))

```
### 2. Split Upload and Polling (Two-step Workflow)

1) Upload only  
```python
upload_meta = client.upload_file("samples/nano_img.png", model_type="AC-1")  
mid = upload_meta["mid"]
```
2) Poll later using the mid
```python
final_media = client.wait_for_media(mid)  
print(final_media["status"], final_media.get("fake"))
```

## Method Reference

### upload_file(path: str, model_type: str) -> Dict[str, Any]
Two-step upload helper.  
Calls POST /media with:  
- name  
- contentType  
- size  
- modelType  

Receives mid and uploadUrl, then uploads the file via PUT to S3.  
Returns media JSON (mid, status, modelType, timestamps, optional srcURL).

### wait_for_media(mid: str, interval: float = 5.0, timeout: float = 600.0) -> Dict[str, Any]
Polling helper.  
Continuously polls GET /media/{mid} until status is:  
- PROCESSED  
- FAILED  
- ERROR  

Sleeps interval seconds between polls.  
Returns final media JSON or raises TimeoutError.

### list_media(**params) -> Dict[str, Any]
Lists media records.  
Wraps GET /media.  
Returns list or paginated response.  
Accepts optional query params (page, pageSize, status).

### process(path: str, model_type: str, interval: float = 5.0, timeout: float = 600.0) -> Dict[str, Any]
High-level method.  
Runs upload_file → wait_for_media.  
Returns processed media JSON (fake, resultURL, heatmapURL, scores).

### get_media(mid: str) -> Dict[str, Any]
Fetch media state.  
Wraps GET /media/{mid}.  
Returns parsed JSON.

### delete_media(mid: str) -> None
Deletes a media entry.  
Wraps DELETE /media/{mid}.  
