# RapidFort Security Advisories

RapidFort provides **validated, research-backed security advisory data** for all RapidFort Curated Images. This dataset enables partner scanners and security platforms to:

- Report vulnerabilities **with full parity to the RapidFort Analyzer**
- Integrate **per-package advisory data** seamlessly into their scanning workflow
- **Eliminate false positives** by excluding vulnerabilities that do not apply to RapidFort Curated Images

The repository contains a structured JSON database of per-package CVE advisories covering **Alpine Linux**, **Ubuntu**, **Debian**, **Red Hat**, **Oracle Linux**, **AlmaLinux**, and **Rocky Linux** distributions.

---

## Repository Structure

```
OS/
├── alpine/    # Alpine Linux advisory files
├── ubuntu/    # Ubuntu advisory files
├── debian/    # Debian advisory files
├── redhat/    # Red Hat advisory files
├── oracle/    # Oracle Linux advisory files
├── alma/      # AlmaLinux advisory files
└── rocky/     # Rocky Linux advisory files
```

Each advisory file covers a single package and follows the naming convention:

```
OS/{os_name}/{package_name}_advisory.json
```

**Examples:**

```
OS/ubuntu/openssl_advisory.json
OS/debian/openssl_advisory.json
OS/alpine/busybox_advisory.json
OS/redhat/zlib_advisory.json
OS/oracle/glibc_advisory.json
OS/alma/ImageMagick-libs_advisory.json
OS/rocky/openssl-libs_advisory.json
```

### Package Name Granularity

The package name used in the filename differs by package format:

| Package Format | Distributions | Keyed By | Example |
|---|---|---|---|
| apk | Alpine Linux | Source package | `openssl_advisory.json` (not `libcrypto3`) |
| dpkg | Ubuntu, Debian | Source package | `glibc_advisory.json` (not `libc6`) |
| rpm | Red Hat, Oracle Linux, AlmaLinux, Rocky Linux | Binary package | `openssl_advisory.json` **and** `openssl-libs_advisory.json` |

Package names prefixed with `rf-` (e.g. `rf-python3`, `rf-chromium`) are RapidFort-rebuilt packages.

---

## Supported Operating Systems

> This list is updated as new OS releases are added to RapidFort Curated Images.

| Distribution | Supported Releases | Package Format |
|---|---|---|
| **Alpine Linux** | 3.20, 3.21, 3.22, 3.23, 3.24 | apk |
| **Ubuntu** | focal (20.04), jammy (22.04), noble (24.04) | dpkg |
| **Debian** | bookworm (12), trixie (13), forky (14) | dpkg |
| **Red Hat** | 5, 6, 7, 8, 9, 10 | rpm |
| **Oracle Linux** | 6, 7, 8, 9, 10 | rpm |
| **AlmaLinux** | 8, 9, 10 | rpm |
| **Rocky Linux** | 8, 9, 10 | rpm |

### Stream Identifiers

Advisory events for every distribution **except Alpine Linux** include an `identifier` field to disambiguate between package streams under the same release key:

| Prefix | Stream | Examples |
|---|---|---|
| `el` | RHEL / CentOS / Oracle Linux / AlmaLinux / Rocky Linux | `el6`, `el7`, `el8`, `el9`, `el10` |
| `fc` | Fedora | `fc39`, `fc40`, `fc41`, `fc42`, `fc43` |
| `ubuntu` | Ubuntu archive | `ubuntu` |
| `debian` | Debian archive | `debian` |
| `rf` | RapidFort-rebuilt package | `rf` |

Which identifiers appear depends on the distribution:

| Distribution | Identifiers Present |
|---|---|
| Alpine Linux | *none — the field is absent* |
| Ubuntu | `ubuntu`, `rf` |
| Debian | `debian`, `rf` |
| Red Hat | `el5`–`el10`, `fc18`–`fc46`, `rf` |
| Oracle Linux | `el2`, `el4`–`el10`, `fc3`–`fc46`, `rf` |
| AlmaLinux, Rocky Linux | `el8`, `el9`, `el10`, `fc18`–`fc46`, `rf` |

---

## Advisory JSON Schema

Each advisory file is a JSON object with the following structure:

```json
{
  "package_name": "zlib",
  "advisory": {
    "<release>": {
      "<CVE-ID>": {
        "cve_id": "CVE-2026-27171",
        "title": "Short vulnerability summary ...",
        "description": "Full vulnerability description.",
        "severity": "MEDIUM",
        "status": "fixed",
        "events": [
          {
            "introduced": "1.3.1-r1",
            "fixed": "1.3.2-r0",
            "identifier": "el9"
          }
        ]
      }
    }
  }
}
```

### Top-Level Fields

| Field | Type | Description |
|---|---|---|
| `package_name` | string | Distribution package name — source package for apk/dpkg, binary package for rpm |
| `advisory` | object | Keyed by OS release identifier (e.g. `"3.21"`, `"22.04"`, `"12"`, `"9"`) |

### CVE Entry Fields

| Field | Type | Description |
|---|---|---|
| `cve_id` | string | CVE identifier (matches the parent object key) |
| `title` | string | Short vulnerability summary |
| `description` | string | Full vulnerability description |
| `severity` | string | One of `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`, `UNKNOWN`, or an empty string |
| `status` | string | `"open"` (no fix available) or `"fixed"` (patch exists) |
| `events` | array | Version range entries describing affected and fixed versions |

### Event Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `introduced` | string | Yes | Version where the vulnerability was introduced. `"0"` means all versions are affected. |
| `fixed` | string | No | Version that resolves the vulnerability. Absent when no fix is available. |
| `identifier` | string | No | **All distributions except Alpine.** Package stream tag (e.g. `el9`, `fc41`, `ubuntu`, `debian`, `rf`) used to disambiguate when a release key maps to multiple package streams. |

### Release Key Format by OS

| OS | Release Key Examples | Description |
|---|---|---|
| Alpine | `"3.20"`, `"3.21"`, `"3.22"`, `"3.23"`, `"3.24"` | Alpine minor version |
| Ubuntu | `"20.04"`, `"22.04"`, `"24.04"` | Ubuntu version number |
| Debian | `"12"`, `"13"`, `"14"` | Major release number |
| Red Hat | `"5"`, `"6"`, `"7"`, `"8"`, `"9"`, `"10"` | Major release number |
| Oracle Linux | `"6"`, `"7"`, `"8"`, `"9"`, `"10"` | Major release number |
| AlmaLinux | `"8"`, `"9"`, `"10"` | Major release number |
| Rocky Linux | `"8"`, `"9"`, `"10"` | Major release number |

---

## Usage Guide

### Step 1: Identify the OS and Release

Determine the operating system and release version of the target system being scanned (e.g. Alpine 3.21, Ubuntu 22.04, Debian 12, Red Hat 9, Oracle Linux 9, AlmaLinux 9, Rocky Linux 9).

### Step 2: Locate the Advisory File

Look up the advisory file for the installed package:

```
OS/{os_name}/{package_name}_advisory.json
```

**Examples:**

```
OS/ubuntu/openssl_advisory.json
OS/debian/dovecot_advisory.json
OS/alpine/busybox_advisory.json
OS/redhat/yelp_advisory.json
OS/oracle/glibc_advisory.json
OS/alma/389-ds-base-libs_advisory.json
OS/rocky/chromium-common_advisory.json
```

For rpm-based distributions, use the binary package name — see [Package Name Granularity](#package-name-granularity).

### Step 3: Load and Navigate the Advisory

Parse the JSON file and navigate to the release key matching your target system:

```
advisory["{release}"] -> dictionary of CVE entries
```

For example, to get all CVEs affecting `zlib` on Ubuntu 22.04:

```
advisory["22.04"] -> { "CVE-2026-27171": { ... } }
```

### Step 4: Evaluate CVEs

For each CVE entry, determine whether it should be reported based on the `status` and version information:

#### Open CVEs

If `status = "open"`, **always report** the vulnerability. No fix is available.

```json
{
  "status": "open",
  "events": [{ "introduced": "0" }]
}
```

#### Fixed CVEs

If `status = "fixed"`, report **only if** the installed version is older than the fixed version:

```
installed_version < fixed_version  -->  report as vulnerable
installed_version >= fixed_version -->  not affected
```

#### Identifier Matching

For every distribution except Alpine, events may include an `identifier` field. When present, **only evaluate events whose `identifier` matches the target system's stream**:

- On RHEL 9, evaluate only events with `identifier = "el9"`
- On Oracle Linux 9, AlmaLinux 9, or Rocky Linux 9, evaluate only events with `identifier = "el9"`
- On Fedora 41, evaluate only events with `identifier = "fc41"`
- On Ubuntu, evaluate only events with `identifier = "ubuntu"` for archive packages, or `identifier = "rf"` for RapidFort-rebuilt packages
- On Debian, evaluate only events with `identifier = "debian"` for archive packages, or `identifier = "rf"` for RapidFort-rebuilt packages

```json
{
  "events": [
    { "introduced": "0", "identifier": "el7" },
    { "introduced": "2:40.3-2.el9", "fixed": "2:40.3-2.el9_6.1", "identifier": "el9" },
    { "introduced": "2:42.2-6.fc41", "fixed": "42.2-9.fc41", "identifier": "fc41" }
  ]
}
```

Ubuntu example — the same CVE tracked separately in the Ubuntu archive and in the RapidFort rebuild, where only the rebuild has a fix:

```json
{
  "events": [
    { "introduced": "0:2.46-3ubuntu2", "identifier": "ubuntu" },
    { "introduced": "0:0", "fixed": "0:2.46-10rfubu", "identifier": "rf" }
  ]
}
```

Debian example — the Debian archive fix, tracked under the `debian` identifier:

```json
{
  "events": [
    { "introduced": "2:3.87.1-1+deb12u1", "fixed": "2:3.87.1-1+deb12u4", "identifier": "debian" }
  ]
}
```

---

## Severity Levels

| Severity | Description |
|---|---|
| `CRITICAL` | Exploitation is straightforward and typically results in system-level compromise |
| `HIGH` | Exploitation could result in significant data loss or service disruption |
| `MEDIUM` | Exploitation requires specific conditions but could impact confidentiality or integrity |
| `LOW` | Limited impact; exploitation is difficult or consequences are minimal |
| `UNKNOWN` | Severity has not been assessed by the upstream source |

---

## Version Formats

Version strings are OS-specific. Consumers must use the appropriate version comparison logic for each distribution.

| OS | Format | Example |
|---|---|---|
| Alpine | `{version}-r{revision}` | `1.3.1-r1` |
| Ubuntu | `{epoch}:{upstream}-{debian}{ubuntu}` | `1:1.2.11.dfsg-2ubuntu9.2` |
| Debian | `{epoch}:{upstream}-{debian}` | `2:3.87.1-1+deb12u4` |
| Red Hat | `{epoch}:{version}-{release}.{dist}` | `2:42.2-5.fc40` |
| Oracle Linux | `{epoch}:{version}-{release}.{dist}` | `2:1.2.11-40.el9` |
| AlmaLinux | `{epoch}:{version}-{release}.{dist}` | `1:2.3.3op2-39.el9_8` |
| Rocky Linux | `{epoch}:{version}-{release}.{dist}` | `0:4.10.0-110.el9_8.2` |

**Notes:**
- The epoch prefix (e.g. `1:`, `2:`) is significant for version ordering and must not be ignored.
- An `introduced` value of `"0"` is a sentinel meaning "all versions from the beginning," not a literal version string. Epoch-qualified forms of the same sentinel also occur — `"0:0"` and, less commonly, values such as `"32:0"` — and carry the same meaning within that epoch.

---

## Schema Variations

A small number of packages (typically third-party or vendor-provided) have minor schema differences:

- The `title` field may be an empty string for packages where the upstream source does not provide a short summary.
- The `severity` field may be `"UNKNOWN"` when no CVSS score is available from the upstream source.
- Debian advisories are occasionally keyed by a Debian Security Advisory identifier rather than a CVE — `DSA-*` (e.g. `DSA-6197-2`) or `DLA-*`. The `cve_id` field still mirrors the parent key. Consumers that assume a `CVE-` prefix should fall back to treating the key as an opaque advisory ID.
