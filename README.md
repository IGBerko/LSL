# LSL - Linux Subsystem for Linux

Images for `lsl`, the [ittyOS](https://github.com/IGBerko/ittyos) tool that runs other Linux
distributions inside a chroot, in the spirit of WSL.

Русская версия — ниже.

## How it works

```
sudo lsl list              # what is installed and what this repository offers
sudo lsl pull debian       # download an image and set it up
sudo lsl run debian        # enter it; "lsl run debian 'apt update'" runs one command
sudo lsl import mylinux file.iso   # use your own ISO / rootfs archive / initramfs
sudo lsl remove debian
```

`lsl pull NAME` reads `index.txt` from this repository (or, if it is missing, lists the
repository folders), downloads the image with curl or wget (no git needed), checks the SHA-256
sum when one is given, and then:

1. mounts the ISO and looks for a root file system: squashfs, `rootfs.img` (Fedora style
   `squashfs -> rootfs.img` layers are followed), or a rootfs tar archive;
2. if there is none, unpacks the initramfs (for example Tiny Core `core.gz`);
3. copies the result to `/var/lib/lsl/distros/NAME/rootfs` and deletes the downloaded image.

`lsl run` mounts `/dev`, `/proc`, `/sys`, copies the DNS settings and does a `chroot`.
It is a chroot, not a container or a virtual machine: there is no init system (no systemd
services), processes and network are shared with the host.

## Adding an image

Plain files in a git repository are limited to 100 MB, so big images go to
**GitHub Releases** (up to 2 GB per file) and this repository only lists them.

1. Create a release (Releases -> Draft a new release), attach the images, publish.
2. Add a line to `index.txt`:

```
# name   path-or-URL                                                               sha256 (optional)
debian   https://github.com/IGBerko/lsl/releases/download/images-1/debian-live.iso  <sha256sum of the file>
tinycore tinycore/corepure64.iso
```

- the second column is either a link, or a path of a file inside this repository;
- the third column is optional: `sha256sum file` prints the value;
- names may contain letters, digits, `.`, `_`, `-`.

Without `index.txt` the repository is read as a listing: every folder is one distribution
(`debian/`, `fedora/` ...) with an image inside, or a single image file in the root.

## What images work

| Image | How it is set up |
|---|---|
| Debian / Ubuntu / Mint live ISO | `live/filesystem.squashfs` or `casper/filesystem.squashfs` |
| Fedora, Void, Mefora style | `LiveOS/squashfs.img` -> `LiveOS/rootfs.img` |
| Arch / Manjaro / Gentoo live ISO | `airootfs.sfs`, `rootfs.sfs`, `image.squashfs` |
| rootfs archive (`.tar.xz`, `.tar.gz`, `.tar.zst`) | unpacked as it is (the smallest option) |
| Tiny Core and similar | no root file system: the initramfs is unpacked |

Net-install ISOs (Debian netinst, ...) have no ready system inside and do not work: use a live
ISO or a rootfs archive. Only x86_64 images run, and only the ones the ittyOS kernel can mount
(squashfs, ext4, erofs, iso9660).

Please publish only what the licence of the distribution allows you to redistribute.

---

## По-русски

Образы для `lsl` — инструмента ittyOS, который запускает другие дистрибутивы Linux в chroot
(как WSL).

```
sudo lsl list              # что установлено и что есть в этом репозитории
sudo lsl pull debian       # скачать образ и подготовить
sudo lsl run debian        # зайти внутрь
sudo lsl import имя файл.iso   # свой ISO / rootfs-архив / initramfs
```

`lsl` ищет в ISO корневую файловую систему (squashfs, rootfs.img, tar-архив). Если её нет,
распаковывает initramfs. Готовая система лежит в `/var/lib/lsl/distros/ИМЯ/rootfs`, вход — через
chroot. Это chroot, а не контейнер: init-системы нет, процессы и сеть общие с хостом.

**Как добавить образ.** Файлы в git ограничены 100 МБ, поэтому большие образы кладите в
**Releases** (до 2 ГБ на файл), а в `index.txt` пишите строку `имя ссылка [sha256]`. Маленькие
(меньше 100 МБ) можно положить прямо в репозиторий и указать путь вместо ссылки.

Образы netinst не подходят (готовой системы внутри нет): берите live-ISO или rootfs-архив.
Выкладывайте только то, что лицензия дистрибутива разрешает распространять.
