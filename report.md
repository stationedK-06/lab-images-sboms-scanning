
OS FILE:
"id": "c02171bf930643dd",
"name": "adduser",
"version": "3.134",
"type": "deb",

Dependency:
"id": "7b992c1572c3e6b0",
"name": "picocolors",
"version": "1.1.1",
"type": "npm",

OS file is from node:20.9.0-bookworm-sliim base image. 
Its from package.json declaration and via npm ci --omit=dev installation process.


### 1. CVE-2023-50387, CVE-2023-50868
- Package: libsystemd0, libudev1
- Installed: 252.17-1~deb12u1
- Fixed in: 252.23-1~deb12u1
- Severity: High
- Type: deb
- Advisory: It is DNSSEC validation flaw (Key Trap) which creates zone forces a resolver to exhuastively check every cross product of DNSKEY and RRSIG. Ultimately causing DOS
- Note: This codebase never does DNS lookup (only listening 0.0.0.0:80800). Thus this vulnerablity is never touched.

### 2. CVE-2024-27983
- Package: node
- Installed: 20.9.0
- Fixed in: 20.12.1 (or 18.20.1 / 21.7.2 on other lines)
- Severity: High
- Type: binary
- Advisory: It cause race condition on Node.js's HTTP/2 HTTP2Session destructor, triggered by CONTINUATION frames with an sudden TCP close.
- Note: this codebase uses node:http not node:http2, thus vulnerability is never touched.

### 3. CVE-2024-2961 
- Package: libc6, libc-bin
- Installed: 2.36-9+deb12u3
- Fixed in: 2.36-9+deb12u6
- Severity: High
- Type: deb
- Advisory: Out of bound write in iconv() from glibc when converting ISO-2022-cn-ext charset. 
- Note: Since npm picocolor only wrap ANSI escape codes, no dependency calls iconv()

### 4. CVE-2026-21710
- Package: node
- Installed: 20.9.0
- Fixed in: 20.20.2
- Severity: High
- Type: binary
- Advisory: __proto__ header causes an uncaught TypeError when application code accessses req.headerDistinct
- Note: Handler only read request.method and .url, Thus this vulnerability is not touched.


What was in the original image? Identify one OS package and one application package from the SBOM, including their names, versions, types, and where they came from.
- OS package adduser 3.134, deb type, bookworm-slim base image
- picocolors 1.1.1 npm. Declared in pacakge.json
What problems did you find and change? Describe the findings you addressed, including the affected packages and advisory identifiers, and explain your Dockerfile changes. Include the relevant Dockerfile diff or changed lines.
- Founding is above. 
- docker file change: node version update, and RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/ to update deb type vuln updates.
What happened afterward? Describe the relevant changes in the final SBOM and scan, any remaining findings, and the smoke-test result. For one package affected by a finding you addressed, connect its source in the original image, its original version, your Dockerfile change, and its final version or absence using evidence from both SBOMs. Explain what happened to the associated scan finding.
- libc6 update from 2.36, CVE-2024-2961 does not exists anymore
- libsystemd0 updated from 252.17-1, and no longer exists
- smoke test failed after the update, I tried to figure out and it seems like it is related to Docker-outside-of Docker networking 127.0.0.1 and host.docker.internal is not connected. (smoke test before version was also failed for some reason in devcontainer.)

```
./scripts/smoke-test lab03:before
[+] Building 0.7s (11/11) FINISHED                                          docker:default
 => [internal] load build definition from Dockerfile                                  0.0s
 => => transferring dockerfile: 509B                                                  0.0s
 => [internal] load metadata for docker.io/library/node:20.20.2-bookworm-slim@sha256  0.0s
 => [internal] load .dockerignore                                                     0.0s
 => => transferring context: 89B                                                      0.0s
 => [1/6] FROM docker.io/library/node:20.20.2-bookworm-slim@sha256:2cf067cfed83d5ea9  0.0s
 => => resolve docker.io/library/node:20.20.2-bookworm-slim@sha256:2cf067cfed83d5ea9  0.0s
 => [internal] load build context                                                     0.0s
 => => transferring context: 127B                                                     0.0s
 => CACHED [2/6] RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lis  0.0s
 => CACHED [3/6] WORKDIR /app                                                         0.0s
 => CACHED [4/6] COPY package.json package-lock.json ./                               0.0s
 => CACHED [5/6] RUN npm ci --omit=dev     && npm cache clean --force     && rm -rf   0.0s
 => CACHED [6/6] COPY src/ ./src/                                                     0.0s
 => exporting to image                                                                0.4s
 => => exporting layers                                                               0.0s
 => => exporting manifest sha256:0b960e0912b514e7490e61ed65cc489345cba98bbda5b92abdd  0.0s
 => => exporting config sha256:bfff48435ea322ff7d3f65ae05e563a06c0dccb7fa47ff9b15d60  0.0s
 => => exporting attestation manifest sha256:0e106f4b0b033d2028827a30f487855609f72a8  0.0s
 => => exporting manifest list sha256:34d7a9b429eeb11fdaf80127eff3b6062d283faa634f3d  0.0s
 => => naming to docker.io/library/lab03:before                                       0.0s
 => => unpacking to docker.io/library/lab03:before                                    0.3s
FAIL: lab03:before did not return HTTP 200 with the expected /health body.
lab03 service listening on 0.0.0.0:8080
node ➜ /workspaces/lab-images-sboms-scanning (main) $ 
```
