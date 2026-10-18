# CS471 Lab 05: Amazon S3 Static Website

**Student:** Ashir Ahmed | **Course:** CS471 Cloud Computing Lab | **Batch:** CS2024

## Architecture
Website files (HTML/CSS/images) -> S3 bucket -> S3 website endpoint -> Browser

- **Bucket:** `cs471-abc-portfolio-2026`
- **Region:** Europe (Stockholm) eu-north-1
- **Objects:** 7 (index.html, about.html, projects.html, style.css, images/profile.svg, images/project-ml.svg, images/project-cloud.svg)
- **Versioning:** Enabled
- **Website endpoint:** http://cs471-abc-portfolio-2026.s3-website.eu-north-1.amazonaws.com

## Deployment
1. Built a three-page site (Home, About, Projects) with a shared `style.css`, navigation on every page, and three images.
2. Created a general purpose bucket with ACLs disabled and Block Public Access left on.
3. Uploaded the HTML and CSS files and the `images` folder, keeping the `images/` prefix.
4. Enabled static website hosting with `index.html` as the index document.
5. Enabled versioning, edited `index.html`, re-uploaded it, and confirmed two versions (current and previous).
6. Reviewed storage classes and the lifecycle rule workflow (no rule created).

## Security and cost notes
- Block all public access was kept **On** because public access was not approved by an instructor. No bucket policy was added.
- Only non-sensitive demo files are in the bucket. No keys, passwords or personal data were uploaded.
- Cost control: checked regional pricing, and old versions and unneeded objects will be removed after assessment.
- Proposed lifecycle rule (not created): move old non-website files to a cheaper storage class after 30 days and delete them after 365 days.
- Storage class: S3 Standard suits website files because they are accessed often and must load immediately. Archive classes need minutes to hours to retrieve, so they cannot serve a live site.

## Troubleshooting
**Problem:** The website endpoint returned `403 Forbidden (AccessDenied)`.
**Diagnosis:** The objects existed and hosting was enabled, but Block Public Access was On and the bucket had no policy, so visitors had no read permission.
**Fix/decision:** Public-read access needs instructor approval, so Block Public Access was left on and the restriction is documented here. With approval, the fix is to turn off Block Public Access for this bucket only and add a bucket policy allowing only `s3:GetObject` on `arn:aws:s3:::cs471-abc-portfolio-2026/*`.

## EC2 + Apache vs S3 Static Website
| Dimension | EC2 + Apache (Lab 04) | S3 Static Website (Lab 05) |
|---|---|---|
| What you manage | OS, Apache, patches, instance and server configuration | Bucket, objects, hosting and access settings |
| Content suitable | Static files and server-side apps | Static HTML/CSS/JS/assets only |
| Operations | Manage instance capacity and web server | No server to administer |
| Security | Security Groups, OS/server config, web permissions | Bucket policy, Block Public Access, IAM, object access |
| Cost factors | Instance runtime, EBS, data transfer, public IPv4 | Storage, requests, data transfer, versions |
| HTTPS/custom domain | Configurable on the server or via other services | Website endpoint is HTTP-only; HTTPS usually via CloudFront and a domain |

## Post-lab answers
**60.** A bucket is a container for objects. An object is a file plus its metadata. The object key is the object's full name, e.g. `images/profile.svg`. A prefix is the shared beginning of keys (e.g. `images/`) that the console shows like a folder.

**61.** Visitors are anonymous browsers, so they need permission to read (`s3:GetObject`) the files or S3 will refuse to serve them.

**62.** An IAM identity policy is attached to a user or role and controls what that identity can do. A bucket policy is attached to the bucket and controls who can access the bucket's objects, including anonymous users.

**63.** Block Public Access protects against accidental public exposure by overriding public ACLs and bucket policies, even if one is set by mistake.

**64.** S3 only serves static files. It cannot run server-side code, databases or backend logic, which EC2 can.

**65.** Versioning keeps old copies, so overwritten or deleted objects can be restored. The cost is that every version is stored and billed.

**66.** For a simple static portfolio I would choose S3. It needs no server maintenance, is cheaper to run, and has less to secure.

**67.** Security: kept Block Public Access on and uploaded only demo files. Cost: checked regional pricing and plan to delete unneeded objects and versions.

## Reflection
This lab showed me that hosting a website does not need a server. S3 was simpler than EC2 because I only managed files and permissions. The 403 error taught me that AWS blocks public access by default, and that public access must be granted deliberately and narrowly. I also learned that versioning is useful for recovery but adds storage cost.
