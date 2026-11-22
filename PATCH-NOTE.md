# facetimehd 0.7.0.1 (distributed source) build fix — MacBook10,1

This branch preserves a build fix for the **distributed 0.7.0.1 source** of the
facetimehd driver (the source installed under `/usr/src/facetimehd-0.7.0.1`,
e.g. via DKMS packages).

## Problem

Building the 0.7.0.1 source against a current kernel (verified with Linux
7.0.0-44) fails in `fthd_isp.c`, function `isp_acpi_set_power()` — the
MacBook10,1-specific ACPI path for the camera sensor power control:

```
fthd_isp.c:114:67: error: 'buffer' undeclared (first use in this function)
fthd_isp.c:118:17: error: 'ret' undeclared (first use in this function)
fthd_isp.c:122:9:  error: 'result' undeclared (first use in this function)
fthd_isp.c:105:17: error: label 'pcie_only' used but not defined
```

## Fix

In `isp_acpi_set_power()`:

- Declare the missing variables:
  - `struct acpi_buffer buffer = { ACPI_ALLOCATE_BUFFER, NULL };`
  - `union acpi_object *result;`
  - `int ret = 0;`
- Define the missing label before `out:`:
  - `pcie_only: return 0;` (fall back to PCIe power management only, as the
    original code intended)

See `facetimehd-0.7.0.2.patch` for the full diff.

## Verified

- Machine: MacBook10,1 (MacBook 12-inch, 2017) running Ubuntu, kernel
  7.0.0-44-generic
- After the fix, the module builds cleanly, was registered in DKMS as
  version 0.7.0.2, and the camera works (`/dev/video0` = "Apple Facetime HD").

## Note

Upstream `patjak/facetimehd` master does **not** have this bug — its
`isp_acpi_set_power()` is written correctly and has no `pcie_only` label.
This fix is only for the 0.7.0.1 distributed source variant, kept here in
case it helps someone with the same build failure.
