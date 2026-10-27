target: https://github.com/eosphoros-ai/DB-GPT
version: 0.8.2

DB-GPT 0.8.2 was discovered to contain a path traversal vulnerability (regression of
CVE-2026-51862) via the /v1/skills/upload endpoint. The filename validation
(_validate_upload_filename, added by PR #3065 and shipped in 0.8.1) was removed in 0.8.2 by the
multi-file upload refactor (PR #3206), allowing arbitrary file write through an absolute-path or
traversal multipart filename. The endpoint is reachable without authentication
(get_user_from_headers returns an admin user by default).

Affected code (agentic_data_api.py, skill_upload):
    filename = file.filename
    tmp_file = upload_dir / filename
    tmp_file.write_bytes(content_bytes)

POC:
```
curl -X POST http://<host>/api/v1/skills/upload \
  -F 'file=@payload.py;filename=/tmp/backdoor.py'
```

The uploaded file is written to /tmp/backdoor.py (outside the pilot/tmp upload directory),
demonstrating arbitrary file write leading to remote code execution.
