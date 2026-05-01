# kelftool

An open-source utility for decrypt, encrypt and sign PS2 KELF and PSX KELF files.

## Build

### Dependencies

- g++ with C++17 support
- OpenSSL 3.0 or older (`libcrypto`)

### Compile

- linux
```bash
make
```

- mac os
> macOS ships without OpenSSL development headers. Install it via Homebrew and set the required flags before building
```bash
brew install openssl
export CPPFLAGS="-I$(brew --prefix openssl)/include"
export LDFLAGS="-L$(brew --prefix openssl)/lib"
make
```

## Usage

```
./build/kelftool.elf <subcommand> <args>
```

### Decrypt

Extract and decrypt content from a KELF file:

```bash
./build/kelftool.elf decrypt <input.kelf> <output.elf>
```

### Encrypt

Pack and sign a raw ELF into a PS2 KELF file:

```bash
./build/kelftool.elf encrypt <headerid> <input.elf> <output.kelf>
```

`<headerid>` specifies the target platform:

| headerid   | Description                                      |
|------------|--------------------------------------------------|
| `fmcb`     | Retail PS2 memory card (Free McBoot)             |
| `dnasload` | Retail PS2 memory card (PSX whitelist bypass)    |
| `fhdb`     | Retail PS2 HDD (HDD OSD / BB Navigator)          |
| `mbr`      | Retail PS2 HDD (MBR injection)                   |
| `dongle`   | Arcade (Namco System 246/256, Konami Python 1)   |

> **Note for `mbr`:** ELF must be headerless and load from address `0x100000`:
> ```bash
> $(EE_OBJCOPY) -O binary -v <input.elf> <headerless.bin>
> ```

#### Optional flags

| Flag                  | Description                                                        |
|-----------------------|--------------------------------------------------------------------|
| `--keys=<section>`    | Key section to use from PS2KEYS.ini (default, retail, dev, arcade, prototype) |
| `--systemtype=<type>` | System type: `PS2` (default) or `PSX`                             |
| `--mgzone=<hex>`      | Region whitelist bitmask, default `0xFF` (all regions)            |
| `--apptype=<hex>`     | Application type, default `1` (xosdmain)                          |
| `--kflags=<value>`    | Header flags: `KELF` (default), `KIRX`, or raw hex value          |

### Validate

Check that all required keys are present in the keystore:

```bash
./build/kelftool.elf validate
./build/kelftool.elf validate --keys=arcade
```

By default validates the `[default]` section. Use `--keys=` to check a specific section. For `arcade` also verifies that `OVERRIDE_KBIT` and `OVERRIDE_KC` are present.

#### Examples

```bash
./build/kelftool.elf encrypt fmcb BOOT.ELF BOOT.KELF
./build/kelftool.elf decrypt BOOT.KELF BOOT.ELF
./build/kelftool.elf encrypt dongle boot.elf boot.bin --keys=arcade --apptype=7
./build/kelftool.elf encrypt fmcb BOOT.ELF BOOT.KELF --mgzone=0x03
```

## Keys

#### You need to bring your own keys.

Place `PS2KEYS.ini` in one of these locations (checked in order):

1. Path set via `PS2KEYS` environment variable
2. `./PS2KEYS.ini` (current working directory)
3. `$HOME/PS2KEYS.ini` (home directory)

See `PS2KEYS_example.ini` for the file format. Actual keys can be found at:
- https://www.psdevwiki.com/ps2/keys
- https://www.psdevwiki.com/ps3/Keys#PS2emu_Keys

## SHA256 Hashes of the keys

### THESE ARE HASHES, NOT THE ACTUAL KEYS

**MG_SIG_MASTER_KEY**=*e6e41172c069b752b9e88d31c70606c580b1c15ee782abd83cf34117bfc47c91*
**MG_SIG_HASH_KEY**=*0dc3a1e225d3e701cfd07c2b25e7a3cc661ded10870218f1f22f936ba350bef5*
**MG_KBIT_MASTER_KEY**=*1512f3f196d6edb723e3c2f4258f6a937c4efd6441785b02d7c9ea7c817ad8fa*
**MG_KBIT_IV**=*14dfe8dbec477884c5eefceb215fa3910e33f4d371ddc125a16ac5ebc9c63a80*
**MG_KC_MASTER_KEY**=*7858c04eb5029d3e7e703ef46829279bfeaf30cb33bc13f54b7f78f0940905c1*
**MG_KC_IV**=*2fa98f860a4562ecb9aff64a79aaeff7c82099c83ca1e61320a9b05f50ca9170*
**MG_ROOTSIG_MASTER_KEY**=*27393c06331f5de238ea62a016f5b4428b11bd2c78d9f0e4bba3bc242a9a1bba*
**MG_ROOTSIG_HASH_KEY**=*5023ea32da5f595d15edf3aad08941dd96ae42a1ad32690a8ca35a024d758bd2*
**MG_CONTENT_TABLE_IV**=*3d9ac39d6e1b69b076da20a38593b2f4ccdd5f943b991c99eacbea13cb1cf0a4*
**MG_CONTENT_IV**=*4e3f5dfaf24c8016c60a23ced78af1e469522dbedb65ca7c8abfb990458f036b*

> For arcade units additional keys `OVERRIDE_KBIT` and `OVERRIDE_KC` are required (use `--keys=arcade`).
> For DTL dev units use `--keys=dev`. Proto keystore probably was never used.
