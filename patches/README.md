# Proxmark5 CEP & T55xx Patches

This directory contains standalone git-formatted patches implementing CEP transport robustness and fast MCU-side ASK demodulation for the Proxmark5.

---

### Patch 0001: PM5: CEP: filter 0xFF poll filler and recover SPI RX overrun (ROERR)
* **File**: `0001-PM5-CEP-filter-0xFF-poll-filler-and-recover-SPI-RX-o.patch`
* **Target File**: `armsrc/pm5_cep.c`
* **Description**:
  1. Drains `0xFF` reply-poll filler bytes clocked on MOSI by the Flipper Zero host while waiting for replies, ensuring `data_available()` does not falsely abort continuous acquisition loops (`WatchEM410x`, `WatchHID`, `SniffIso14443a`).
  2. Clears `SPI_I2S_ROERR_FLAG` (RX overrun) that occurs when long blocking operations (e.g. LF `DoAcquisition_default`) freeze the `SPI_DT` register, enabling subsequent break-loop commands (`0x55`) to be detected.
  3. Flushes pre-sync trailing `0xFF` poll bytes prior to `CEP_SYNC_BYTE` (`0x55`) in `cep_spi_data_available()`.
  4. Flushes `SPI_I2S_RDBF_FLAG` and clears `ROERR` in `cep_spi_write()` cleanup.

### Patch 0002: LF: T55xx: add MCU-side ASK demodulation to T55xxReadBlock
* **File**: `0002-LF-T55xx-add-MCU-side-ASK-demodulation-to-T55xxReadB.patch`
* **Target File**: `armsrc/lfops.c`
* **Description**:
  1. Runs `askdemod()` directly on the acquired raw samples in `BigBuf` during `T55xxReadBlock` (when `brute_mem == false`).
  2. Detects identical adjacent 32-bit repeating blocks (at 32 or 33 bit intervals).
  3. Returns the decoded 32-bit `uint32_t` block directly in the `CMD_LF_T55XX_READBL` response payload, enabling instantaneous block reads across constrained serial/SPI links without requiring raw trace downloads.
  4. Preserves backward compatibility: if demodulation fails, returns an empty payload as before.

---

### Applying Patches
```bash
git am patches/0001-PM5-CEP-filter-0xFF-poll-filler-and-recover-SPI-RX-o.patch
git am patches/0002-LF-T55xx-add-MCU-side-ASK-demodulation-to-T55xxReadB.patch
```
