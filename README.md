# SISOP-4-2024-MH-IT23
1. Waktu pengerjaan dimulai hari Kamis (16 Mei 2024) hingga hari Selasa (21 Mei 2024) pukul 23.59 WIB.
2. Praktikan diharapkan membuat laporan penjelasan dan penyelesaian soal dalam bentuk Readme(github).
3. Format nama repository github “Sisop-[Nomor Modul]-2023-[Kode Dosen Kelas]-[Nama Kelompok]” (contoh:Sisop-4-2024-MH-IT01).
4. Struktur repository seperti berikut:
			—soal_1:
				— inikaryakita.c
                                       
      —soal_2:
	      — pastibisa.c

      —soal_3:
				— archeology.c
				

	Jika melanggar struktur repo akan dianggap sama dengan curang dan menerima konsekuensi sama dengan melakukan kecurangan.
6. Setelah pengerjaan selesai, semua script bash, awk, dan file yang berisi cron job ditaruh di github masing - masing kelompok, dan link github diletakkan pada form yang disediakan. Pastikan github di setting ke publik.
7. Commit terakhir maksimal 10 menit setelah waktu pengerjaan berakhir. Jika melewati maka akan dinilai berdasarkan commit terakhir.
8. Jika tidak ada pengumuman perubahan soal oleh asisten, maka soal dianggap dapat diselesaikan.
9. Jika ditemukan soal yang tidak dapat diselesaikan, harap menuliskannya pada Readme beserta permasalahan yang ditemukan.
10. Praktikan tidak diperbolehkan menanyakan jawaban dari soal yang diberikan kepada asisten maupun praktikan dari kelompok lainnya.
11. Jika ditemukan indikasi kecurangan dalam bentuk apapun di pengerjaan soal shift, maka nilai dianggap 0.
12. Pengerjaan soal shift sesuai dengan modul yang telah diajarkan.
13. Zip dari repository dikirim ke email asisten penguji dengan subjek yang sama dengan nama judul repository, dikirim sebelum deadline dari soal shift
14. Jika terdapat revisi soal akan dituliskan pada halaman terakhir

### SOAL 1
1. fungsi untuk menambah watermark
```
#define FUSE_USE_VERSION 30

#include <fuse.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <dirent.h>
#include <errno.h>

static const char *gallery_path = "/home/aceen/testing/portofolio/gallery";
static const char *wm_folder = "/home/aceen/testing/portofolio/gallery/wm.";
static const char *bahaya_path = "/home/aceen/testing/portofolio/bahaya";
static const char *script_file = "/home/aceen/testing/portofolio/bahaya/script.sh";

void add_watermark(const char *input, const char *output) {
    char command[1024];
    sprintf(command, "convert %s -gravity South -annotate 0 'inikaryakita.id' %s", input, output);
    system(command);
}
```
fungsi untuk reverse file
```
void reverse_file_content(const char *input, const char *output) {
    FILE *in = fopen(input, "r");
    if (in == NULL) {
        perror("fopen input");
        return;
    }

    fseek(in, 0, SEEK_END);
    long fsize = ftell(in);
    fseek(in, 0, SEEK_SET);

    char *content = malloc(fsize + 1);
    fread(content, 1, fsize, in);
    fclose(in);
    content[fsize] = '\0';

    FILE *out = fopen(output, "w");
    if (out == NULL) {
        perror("fopen output");
        free(content);
        return;
    }

    for (long i = fsize - 1; i >= 0; i--) {
        fputc(content[i], out);
    }

    fclose(out);
    free(content);
}
```
fungsi fuse
```
static int xmp_getattr(const char *path, struct stat *stbuf, struct fuse_file_info *fi) {
    (void) fi;
    int res;
    char fpath[1000];
    
    sprintf(fpath, ".%s", path);
    res = lstat(fpath, stbuf);

    if (res == -1) return -errno;

    return 0;
}

static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi, enum fuse_readdir_flags flags) {
    (void) offset;
    (void) fi;
    (void) flags;

    DIR *dp;
    struct dirent *de;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    dp = opendir(fpath);
    if (dp == NULL) return -errno;

    while ((de = readdir(dp)) != NULL) {
        struct stat st;
        memset(&st, 0, sizeof(st));
        st.st_ino = de->d_ino;
        st.st_mode = de->d_type << 12;
        if (filler(buf, de->d_name, &st, 0, 0)) break;
    }

    closedir(dp);
    return 0;
}

static int xmp_open(const char *path, struct fuse_file_info *fi) {
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    res = open(fpath, fi->flags);
    if (res == -1) return -errno;

    close(res);
    return 0;
}

static int xmp_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi) {
    int fd;
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    fd = open(fpath, O_RDONLY);
    if (fd == -1) return -errno;

    res = pread(fd, buf, size, offset);
    if (res == -1) res = -errno;

    close(fd);
    return res;
}

static int xmp_write(const char *path, const char *buf, size_t size, off_t offset, struct fuse_file_info *fi) {
    int fd;
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    fd = open(fpath, O_WRONLY);
    if (fd == -1) return -errno;

    res = pwrite(fd, buf, size, offset);
    if (res == -1) res = -errno;

    close(fd);
    return res;
}

static int xmp_rename(const char *from, const char *to, unsigned int flags) {
    int res;
    char ffrom[1000];
    char fto[1000];

    if (flags) return -EINVAL;

    sprintf(ffrom, ".%s", from);
    sprintf(fto, ".%s", to);

    if (strncmp(fto, wm_folder, strlen(wm_folder)) == 0) {
        // watermark
        add_watermark(ffrom, fto);
        unlink(ffrom); // rm file ori
    } else if (strncmp(fto, bahaya_path, strlen(bahaya_path)) == 0 && strncmp(fto + strlen(bahaya_path) + 1, "test", 4) == 0) {
        // reverse
        reverse_file_content(ffrom, fto);
        unlink(ffrom); // rm file ori
    } else {
        res = rename(ffrom, fto);
        if (res == -1) return -errno;
    }

    return 0;
}

static int xmp_chmod(const char *path, mode_t mode, struct fuse_file_info *fi) {
    (void) fi;
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);

    if (strcmp(fpath, script_file) == 0) {
        mode |= S_IXUSR | S_IXGRP | S_IXOTH;
    }

    res = chmod(fpath, mode);
    if (res == -1) return -errno;

    return 0;
}

static int xmp_readlink(const char *path, char *buf, size_t size) {
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    res = readlink(fpath, buf, size - 1);
    if (res == -1) return -errno;

    buf[res] = '\0';
    return 0;
}

static int xmp_mknod(const char *path, mode_t mode, dev_t rdev) {
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    res = mknod(fpath, mode, rdev);
    if (res == -1) return -errno;

    return 0;
}

static int xmp_unlink(const char *path) {
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    res = unlink(fpath);
    if (res == -1) return -errno;

    return 0;
}

static int xmp_mkdir(const char *path, mode_t mode) {
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    res = mkdir(fpath, mode);
    if (res == -1) return -errno;

    return 0;
}

static int xmp_rmdir(const char *path) {
    int res;
    char fpath[1000];

    sprintf(fpath, ".%s", path);
    res = rmdir(fpath);
    if (res == -1) return -errno;

    return 0;
}

static int xmp_symlink(const char *from, const char *to) {
    int res;
    char ffrom[1000];
    char fto[1000];

    sprintf(ffrom, ".%s", from);
    sprintf(fto, ".%s", to);
    res = symlink(ffrom, fto);
    if (res == -1) return -errno;

    return 0;
}

static struct fuse_operations xmp_oper = {
    .getattr    = xmp_getattr,
    .readdir    = xmp_readdir,
    .open       = xmp_open,
    .read       = xmp_read,
    .write      = xmp_write,
    .rename     = xmp_rename, 
    .chmod      = xmp_chmod,
    .readlink   = xmp_readlink,
    .mknod      = xmp_mknod,
    .unlink     = xmp_unlink,
    .mkdir      = xmp_mkdir,
    .rmdir      = xmp_rmdir,
    .symlink    = xmp_symlink,
};

```
fungsi main yang akan memanggil fungsi fuse
```
int main(int argc, char *argv[]) {
    struct stat st;
    
    if (stat(wm_folder, &st) == -1) {
        mkdir(wm_folder, 0700);
    }

    printf("Gallery path: %s\n", gallery_path);

    return fuse_main(argc, argv, &xmp_oper, NULL);
}

```

Error : <br />
ketika program dimount ke fuse, tidak ada fungsi yang berhasil berjalan dengan benar. Watermark tidak muncul pada gambar, isi file test juga tidak di reverse. <br />
REVISI <br />
```
#define FUSE_USE_VERSION 31  
#include <fuse.h>            
#include <errno.h>           
#include <dirent.h>          
#include <stdio.h>           
#include <stdlib.h>          
#include <string.h>          
#include <sys/stat.h>        
#include <sys/types.h>       
#include <unistd.h>          

#define MAX_PATH_LENGTH 512 

void apply_watermark(const char *src_path, const char *dst_path) {
    char command[1024];  
    snprintf(command, sizeof(command), "convert \"%s\" -gravity south -pointsize 36 -fill white -draw \"text 0,0 'inikaryakita.id'\" \"%s\"",
             src_path, dst_path);
    system(command); 
}

void reverse_file_content(const char *src_path, const char *dst_path) {
    FILE *file = fopen(src_path, "r");  
    // Membuka file sumber untuk membaca
    FILE *outputFile = fopen(dst_path, "w");  
    // Membuka file tujuan untuk write

    if (file == NULL || outputFile == NULL) {  
        // Memeriksa eror
        perror("fopen");
        return;
    }

    fseek(file, 0, SEEK_END);  
    long fileSize = ftell(file);  
    fseek(file, 0, SEEK_SET);  

    char *content = malloc(fileSize + 1);  
    fread(content, 1, fileSize, file);  
    content[fileSize] = '\0';  

    for (long i = fileSize - 1; i >= 0; i--) {  
        fputc(content[i], outputFile);
    }

    free(content);  
    fclose(file);  
    fclose(outputFile);  
}

static int do_getattr(const char *path, struct stat *st) {
    memset(st, 0, sizeof(struct stat));  
    if (strcmp(path, "/") == 0 || strcmp(path, "/gallery") == 0 || strcmp(path, "/bahaya") == 0) {
        st->st_mode = S_IFDIR | 0755;  
        st->st_nlink = 2;  
    } else {
        st->st_mode = S_IFREG | 0644;  
        st->st_nlink = 1;
        st->st_size = 1024;
    }
    return 0;  
}

static int do_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi) {
    (void)offset;  
    (void)fi;      

    if (strcmp(path, "/") == 0) {  
        filler(buf, ".", NULL, 0);  
        filler(buf, "..", NULL, 0);  
        filler(buf, "gallery", NULL, 0);  
        filler(buf, "bahaya", NULL, 0);  
    } else if (strcmp(path, "/gallery") == 0 || strcmp(path, "/bahaya") == 0) { 
        DIR *dp;
        struct dirent *de;
        if ((dp = opendir(path)) == NULL) { 
            return -errno;
        }
        while ((de = readdir(dp)) != NULL) {  
            filler(buf, de->d_name, NULL, 0);  
        }
        closedir(dp);  
    }
    return 0;  
}

static int do_open(const char *path, struct fuse_file_info *fi) {
    return 0;  
}

static int do_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi) {
    (void)fi;  

    char src_path[MAX_PATH_LENGTH];  
    char temp_path[MAX_PATH_LENGTH];  
    snprintf(src_path, sizeof(src_path), ".%s", path);  

    if (strstr(path, "/gallery/") == path) {  
        snprintf(temp_path, sizeof(temp_path), "/tmp/watermarked%s", path + strlen("/gallery"));  
        apply_watermark(src_path, temp_path);  
        FILE *file = fopen(temp_path, "r");  
        if (file == NULL) {
            return -errno;  
        }
        fseek(file, offset, SEEK_SET);  
        size_t bytes_read = fread(buf, 1, size, file);  
        fclose(file);  
        return bytes_read;  
        snprintf(temp_path, sizeof(temp_path), "/tmp/reversed%s", path + strlen("/bahaya"));  
        reverse_file_content(src_path, temp_path);  
        FILE *file = fopen(temp_path, "r");  
        if (file == NULL) {
            return -errno;  
        }
        fseek(file, offset, SEEK_SET);  
        size_t bytes_read = fread(buf, 1, size, file);  
        fclose(file);  
        return bytes_read;  
    }
    return -ENOENT;  
    
}

static struct fuse_operations operations = {
    .getattr = do_getattr,  
    .readdir = do_readdir,  
    .open = do_open,        
    .read = do_read,        
};

int main(int argc, char *argv[]) {
    mkdir("gallery", 0755); 
    mkdir("bahaya", 0755);
    return fuse_main(argc, argv, &operations, NULL);  
}
```


### SOAL 2

```
#define FUSE_USE_VERSION 29
#include <fuse.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <fcntl.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>
#include <dirent.h>
#include <ctype.h>

// Variabel global
static const char *base_dir = "/home/syahrhm/sisop4/sensitif";
static const char *log_file = "/home/syahrhm/sisop4/logs-fuse.log";
static const char *password = "bisatapimati";

// Deklarasi fungsi
void createLog(const char *status, const char *tag, const char *information);
int checkPass();
void decode_base64(const char *input, char *output);
void decode_rot13(const char *input, char *output);
void decode_hex(const char *input, char *output);
void reverse(const char *input, char *output);
static int xmp_getattr(const char *path, struct stat *stbuf);
static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi);
static int xmp_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi);

static struct fuse_operations xmp_oper = {
    .getattr = xmp_getattr,
    .readdir = xmp_readdir,
    .read = xmp_read,
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <mountpoint>\n", argv[0]);
        return 1;
    }
    return fuse_main(argc, argv, &xmp_oper, NULL);
}

void createLog(const char *status, const char *tag, const char *information) {
    FILE *log = fopen(log_file, "a");
    if (log == NULL) return;

    time_t now = time(NULL);
    struct tm *t = localtime(&now);

    fprintf(log, "[%s]::%02d/%02d/%04d-%02d:%02d:%02d::[%s]::[%s]\n",
            status, t->tm_mday, t->tm_mon + 1, t->tm_year + 1900,
            t->tm_hour, t->tm_min, t->tm_sec, tag, information);
    fclose(log);
}

int checkPass() {
    char input[100];
    printf("Enter password: ");
    scanf("%99s", input);
    if (strcmp(input, password) == 0) {
        createLog("SUCCESS", "checkPass", "Password correct");
        return 1;
    } else {
        createLog("FAILED", "checkPass", "Password incorrect");
        return 0;
    }
}

void decode_base64(const char *input, char *output) {
    const char b64_table[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    int input_len = strlen(input);
    int i = 0, j = 0;
    int output_len = 0;

    unsigned char a3[3];
    unsigned char a4[4];

    while (input_len--) {
        if (*input == '=') {
            break;
        }
        a4[i++] = *(input++);
        if (i == 4) {
            for (i = 0; i < 4; i++) {
                a4[i] = strchr(b64_table, a4[i]) - b64_table;
            }

            a3[0] = (a4[0] << 2) | (a4[1] >> 4);
            a3[1] = ((a4[1] & 15) << 4) | (a4[2] >> 2);
            a3[2] = ((a4[2] & 3) << 6) | a4[3];

            for (i = 0; (i < 3); i++) {
                output[output_len++] = a3[i];
            }
            i = 0;
        }
    }

    if (i) {
        for (j = i; j < 4; j++) {
            a4[j] = '\0';
        }

        for (j = 0; j < 4; j++) {
            a4[j] = strchr(b64_table, a4[j]) - b64_table;
        }

        a3[0] = (a4[0] << 2) | (a4[1] >> 4);
        a3[1] = ((a4[1] & 15) << 4) | (a4[2] >> 2);
        a3[2] = ((a4[2] & 3) << 6) | a4[3];

        for (j = 0; (j < i - 1); j++) {
            output[output_len++] = a3[j];
        }
    }
    output[output_len] = '\0';
}

void decode_rot13(const char *input, char *output) {
    for (int i = 0; input[i] != '\0'; i++) {
        char c = input[i];
        if (c >= 'a' && c <= 'z') {
            output[i] = (c - 'a' + 13) % 26 + 'a';
        } else if (c >= 'A' && c <= 'Z') {
            output[i] = (c - 'A' + 13) % 26 + 'A';
        } else {
            output[i] = c;
        }
    }
    output[strlen(input)] = '\0';
}

void decode_hex(const char *input, char *output) {
    int len = strlen(input) / 2;
    for (int i = 0; i < len; i++) {
        sscanf(input + 2 * i, "%2hhx", &output[i]);
    }
    output[len] = '\0';
}

void reverse(const char *input, char *output) {
    int len = strlen(input);
    for (int i = 0; i < len; i++) {
        output[i] = input[len - i - 1];
    }
    output[len] = '\0';
}

static int xmp_getattr(const char *path, struct stat *stbuf) {
    char fpath[1000];
    sprintf(fpath, "%s%s", base_dir, path);
    int res = lstat(fpath, stbuf);
    if (res == -1) return -errno;
    return 0;
}

static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi) {
    DIR *dp;
    struct dirent *de;

    char fpath[1000];
    sprintf(fpath, "%s%s", base_dir, path);

    if (strncmp(path, "/rahasia", 8) == 0 && !checkPass()) {
        return -EACCES;
    }

    dp = opendir(fpath);
    if (dp == NULL) return -errno

;

    while ((de = readdir(dp)) != NULL) {
        struct stat st;
        memset(&st, 0, sizeof(st));
        st.st_ino = de->d_ino;
        st.st_mode = de->d_type << 12;
        if (filler(buf, de->d_name, &st, 0)) break;
    }
    closedir(dp);
    return 0;
}

static int xmp_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi) {
    char fpath[1000];
    char decoded[1000];
    sprintf(fpath, "%s%s", base_dir, path);

    if (strncmp(path, "/rahasia", 8) == 0 && !checkPass()) {
        return -EACCES;
    }

    int fd = open(fpath, O_RDONLY);
    if (fd == -1) return -errno;

    int res = pread(fd, buf, size, offset);
    if (res == -1) res = -errno;

    close(fd);

    if (strncmp(path, "/pesan/base64_", 14) == 0) {
        buf[res] = '\0';  // Ensure the buffer is null-terminated
        decode_base64(buf, decoded);
        strcpy(buf, decoded);
        res = strlen(decoded);
    } else if (strncmp(path, "/pesan/rot13_", 13) == 0) {
        buf[res] = '\0';  // Ensure the buffer is null-terminated
        decode_rot13(buf, decoded);
        strcpy(buf, decoded);
        res = strlen(decoded);
    } else if (strncmp(path, "/pesan/hex_", 11) == 0) {
        buf[res] = '\0';  // Ensure the buffer is null-terminated
        decode_hex(buf, decoded);
        strcpy(buf, decoded);
        res = strlen(decoded);
    } else if (strncmp(path, "/pesan/rev_", 11) == 0) {
        buf[res] = '\0';  // Ensure the buffer is null-terminated
        reverse(buf, decoded);
        strcpy(buf, decoded);
        res = strlen(decoded);
    }

    return res;
};
```

### Soal 3
[archeology.c]
```
#define FUSE_USE_VERSION 31

#include <fuse.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <dirent.h>
#include <errno.h>
#include <fcntl.h>
#include <stddef.h>
#include <assert.h>
#include <sys/stat.h>
#include <time.h>

#define MAX_BUFFER 1028
#define MAX_SPLIT 10000
static const char *source_path = "/home/marcell/Desktop/sisop/modul4/soal3/relics";
```
Melakukan define terhadap version fuse yang akan digunakan, library yang akan digunakan, size buffer dan maximum size dari fragment file pada relics yaitu 10 Kb serta melakukan spesifikasi base directory dimana file fragment akan disimpan
```
int fuze_getattr(const char *path, struct stat *statbuf)
{
    printf("Entering fuze_getattr for %s\n", path);
    memset(statbuf, 0, sizeof(struct stat));
    statbuf->st_uid = getuid();
    statbuf->st_gid = getgid();
    statbuf->st_atime = time(NULL);
    statbuf->st_mtime = time(NULL);
    if (strcmp(path, "/") == 0)
    {
        statbuf->st_mode = S_IFDIR | 0755;
        statbuf->st_nlink = 2;
        printf("Leaving fuze_getattr for /\n");
        return 0;
    }
    char full_path[MAX_BUFFER];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s\n", full_path);

    statbuf->st_mode = S_IFREG | 0644;
    statbuf->st_nlink = 1;
    statbuf->st_size = 0;
    char apath[MAX_BUFFER + 4];
    FILE *fd;
    int i = 0;
    while (1)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, i++);
        printf("Checking fragment: %s\n", apath);
        fd = fopen(apath, "rb");
        if (!fd)
        {
            if (i == 1)
            {
                printf("File not found: %s\n", apath);
                printf("Leaving fuze_getattr with error for %s\n", path);
                return -ENOENT;
            }
            break;
        }
        fseek(fd, 0L, SEEK_END);
        statbuf->st_size += ftell(fd);
        fclose(fd);
    }
    printf("File size: %ld\n", statbuf->st_size);
    printf("Leaving fuze_getattr for %s\n", path);
    return 0;
}
```
Membuat fungsi getattr dimana akan menghandle pengambilan atribut file dengan parameter path dan statbuf dengan cara menginisialisasi statbuf dengan UID dan GID, akses dan waktu modifikasi, cek apakah path root dan set mode directory serta menghitung link.

```
int fuze_readdir(const char *path, void *buffer, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi)
{
    printf("Entering fuze_readdir for %s\n", path);
    (void)fi;
    (void)offset;
    filler(buffer, ".", NULL, 0);
    filler(buffer, "..", NULL, 0);
    char full_path[MAX_BUFFER];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s\n", full_path);

    DIR *dirpath = opendir(full_path);
    struct dirent *de;
    struct stat st;
    if (!dirpath)
    {
        printf("Could not open directory: %s\n", full_path);
        printf("Leaving fuze_readdir with error for %s\n", path);
        return -errno;
    }
    while ((de = readdir(dirpath)) != NULL)
    {
        memset(&st, 0, sizeof(st));
        st.st_ino = de->d_ino;
        st.st_mode = de->d_type << 12;
        if (strstr(de->d_name, ".000") == NULL)
            continue;
        char kats[MAX_BUFFER];
        strcpy(kats, de->d_name);
        kats[strlen(kats) - 4] = '\0';
        printf("Adding %s to directory listing\n", kats);
        if (filler(buffer, kats, &st, 0))
            break;
    }
    closedir(dirpath);
    printf("Leaving fuze_readdir for %s\n", path);
    return 0;
}
```
Membuat fungsi readdir dimana akan membaca isi dari sebuah directory dengan parameter path, buffer, filler, offset. Berfungsi untuk melakukan listing terhadap sebuah directory dan menghapus '.000' lalu memasukkannya ke listing directory
```
int fuze_read(const char *path, char *buffer, size_t sz, off_t os, struct fuse_file_info *fi)
{
    printf("Entering fuze_read for %s\n", path);
    char full_path[MAX_BUFFER];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s, size: %zu, offset: %ld\n", full_path, sz, os);

    char apath[MAX_BUFFER + 4];
    FILE *fdir;
    int i = 0;
    size_t size_read;
    size_t sum_read = 0;
    while (sz > 0)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, i++);
        printf("Reading from fragment: %s\n", apath);
        fdir = fopen(apath, "rb");
        if (!fdir)
        {
            printf("Could not open fragment: %s\n", apath);
            break;
        }
        fseek(fdir, 0L, SEEK_END);
        size_t sz_part = ftell(fdir);
        fseek(fdir, 0L, SEEK_SET);
        if (os >= sz_part)
        {
            printf("Skipping %s due to offset\n", apath);
            os -= sz_part;
            fclose(fdir);
            continue;
        }
        fseek(fdir, os, SEEK_SET);
        size_read = fread(buffer, 1, sz, fdir);
        printf("Read %zu bytes from %s\n", size_read, apath);
        fclose(fdir);
        buffer += size_read;
        sz -= size_read;
        sum_read += size_read;
        os = 0;
    }
    printf("Leaving fuze_read for %s, total %zu bytes read\n", path, sum_read);
    return sum_read;
}
```
Membuat fungsi read dimana akan membaca data dari sebuah file dengan parameter path, buffer, sz, os. Berfungsi untuk membaca data dari fragment file yang benar berdasarkan ukuran dan offset file tersebut dengan cara mencari dan menentukan fragment yang akan dibaca berdasarkan offset serta membaca data yang diperlukan dari setiap fragment sampai ukuran yang diinginkan tercapai.
```
int fuze_write(const char *path, const char *buffer, size_t sz, off_t os, struct fuse_file_info *fi)
{
    printf("Entering fuze_write for %s\n", path);
    char full_path[MAX_BUFFER];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s, size: %zu, offset: %ld\n", full_path, sz, os);

    char apath[MAX_BUFFER + 4];
    FILE *fdir;
    int pcur = os / MAX_SPLIT;
    size_t pos = os % MAX_SPLIT;
    size_t sum_write = 0;
    while (sz > 0)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, pcur++);
        printf("Writing to fragment: %s\n", apath);
        fdir = fopen(apath, "r+b");
        if (!fdir)
        {
            fdir = fopen(apath, "wb");
            if (!fdir)
            {
                printf("Could not create fragment: %s\n", apath);
                printf("Leaving fuze_write with error for %s\n", path);
                return -errno;
            }
        }
        fseek(fdir, pos, SEEK_SET);
        size_t sz_write = (sz > (MAX_SPLIT - pos)) ? (MAX_SPLIT - pos) : sz;
        printf("Writing %zu bytes to %s\n", sz_write, apath);
        fwrite(buffer, 1, sz_write, fdir);
        fclose(fdir);
        buffer += sz_write;
        sz -= sz_write;
        sum_write += sz_write;
        pos = 0;
    }
    printf("Leaving fuze_write for %s, total %zu bytes written\n", path, sum_write);
    return sum_write;
}
```
Membuat fungsi write dimana akan menulis data dari sebuah file dengan parameter, path, buffer, sz, os. Berfungsi untuk menulis data ke fragment file dan membuat fragment baru bila diperlukan dengan cara mencari serta menentukan fragment yang akan ditulis berdasarkan offset serta menulis data ke fragment yang sesuai dan membuatnya apabila belum ada.
```
int fuze_create(const char *path, mode_t md, struct fuse_file_info *fi)
{
    printf("Entering fuze_create for %s\n", path);
    (void)fi;
    char full_path[MAX_BUFFER];
    snprintf(full_path, MAX_BUFFER, "%s%s.000", source_path, path);
    printf("Creating file: %s\n", full_path);

    int rez = creat(full_path, md);
    if (rez == -1)
    {
        printf("Could not create file: %s\n", full_path);
        printf("Leaving fuze_create with error for %s\n", path);
        return -errno;
    }
    close(rez);
    printf("Leaving fuze_create for %s\n", path);
    return 0;
}
```
Membuat fungsi create dimana akan membuat file baru dengan parameter path, md. Berfungsi untuk membuat file baru dengan fragment '.000' dengan cara membuat path untuk fragment pertama yaitu '.000' dan membuat filenya.
```
int fuze_unlink(const char *path)
{
    printf("Entering fuze_unlink for %s\n", path);
    char full_path[MAX_BUFFER];
    char apath[MAX_BUFFER + 4];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s\n", full_path);

    int pcur = 0;
    while (1)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, pcur++);
        printf("Unlinking fragment: %s\n", apath);
        int rez = unlink(apath);
        if (rez == -1)
        {
            if (errno == ENOENT)
                break;
            printf("Could not unlink fragment: %s\n", apath);
            printf("Leaving fuze_unlink with error for %s\n", path);
            return -errno;
        }
    }
    printf("Leaving fuze_unlink for %s\n", path);
    return 0;
}
```
Membuat fungsi unlink untuk menghapus sebuah file dengan parameter path. Berfungsi untuk menghapus seluruh fragment dari sebuah file dengan cara melakukan loop terhadap seluruh fragment dan menghapus setiap fragment file sampai tidak ada lagi fragment file.
```
int fuze_truncate(const char *path, off_t sz)
{
    printf("Entering fuze_truncate for %s\n", path);
    char full_path[MAX_BUFFER];
    char apath[MAX_BUFFER + 4];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s, size: %ld\n", full_path, sz);

    int pcurr_t = 0;
    size_t sz_part;
    off_t sz_rmn = sz;
    while (sz_rmn > 0)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, pcurr_t++);
        if (sz_rmn > MAX_SPLIT)
            sz_part = MAX_SPLIT;
        else
            sz_part = sz_rmn;
        printf("Truncating fragment: %s to size: %zu\n", apath, sz_part);
        int rez = truncate(apath, sz_part);
        if (rez == -1)
        {
            printf("Could not truncate fragment: %s\n", apath);
            printf("Leaving fuze_truncate with error for %s\n", path);
            return -errno;
        }
        sz_rmn -= sz_part;
    }
    int pcurr_u = 0;
    while (1)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, pcurr_u++);
        int rez = unlink(apath);
        if (rez == -1)
        {
            if (errno == ENOENT)
                break;
            printf("Could not unlink fragment: %s\n", apath);
            printf("Leaving fuze_truncate with error for %s\n", path);
            return -errno;
        }
    }
    printf("Leaving fuze_truncate for %s\n", path);
    return 0;
}
```
Membuat file truncate untuk mengurangi size sebuah file ke size yang diinginkan dengan parameter path dan sz. Berfungsi untuk mengurangi atau menambah size file dan menyesuaikan fragment file terhadap size file.
```
int fuze_utimens(const char *path, const struct timespec tspec[2])
{
    printf("Entering fuze_utimens for %s\n", path);
    char full_path[MAX_BUFFER];
    snprintf(full_path, MAX_BUFFER, "%s%s", source_path, path);
    printf("Full path: %s\n", full_path);

    char apath[MAX_BUFFER + 4];
    int pcurr = 0;
    while (1)
    {
        snprintf(apath, MAX_BUFFER + 4, "%s.%03d", full_path, pcurr++);
        printf("Updating timestamps for fragment: %s\n", apath);
        int res = utimensat(AT_FDCWD, apath, tspec, 0);
        if (res == -1)
        {
            if (errno == ENOENT)
                break;
            printf("Could not update timestamps for fragment: %s\n", apath);
            printf("Leaving fuze_utimens with error for %s\n", path);
            return -errno;
        }
    }
    printf("Leaving fuze_utimens for %s\n", path);
    return 0;
}
```
Membuat file utimens dimana akan mengupdate timestamp dari sebuah file dengan parameter path dan timespec. Berfunsi untuk menentukan access serta modifikasi waktu ke seluruh fragment file dengan cara melakukan loop terhadap keseluruhan fragment lalu melakukan update ke timestamps keseluruhan fragment file.
```
int main(int argc, char *argv[])
{
    struct fuse_operations fuze_oper = {
        .getattr = fuze_getattr,
        .readdir = fuze_readdir,
        .read = fuze_read,
        .write = fuze_write,
        .create = fuze_create,
        .unlink = fuze_unlink,
        .truncate = fuze_truncate,
        .utimens = fuze_utimens,
    };
    printf("Starting FUSE filesystem\n");
    return fuse_main(argc, argv, &fuze_oper, NULL);
}
```
Pada fungsi main kita melakukan inisialisasi FUSE dan memulai filesystemnya dengan operasi yang telah kita buat dengan cara mendefinisikan sebuah struktu 'fuse_operations' dengan pointer ke functions yang akan diimplementasikan lalu memanggil 'fuse_main' untuk memulai filesystem FUSE.

Listing directory [nama_bebas]
![image](https://github.com/v0rein/SISOP-4-2024-MH-IT23/assets/143814923/fae936fe-0124-4918-b2ab-6fb1aed78073)

Hasil copy ke directory apapun dari [nama_bebas] dan memasukkan file ke [nama_bebas] lalu cek di directory relics
![image](https://github.com/v0rein/SISOP-4-2024-MH-IT23/assets/143814923/6dbea102-8c6e-4d33-9fa5-a96fa980eb04)

Delete file di [nama_bebas] maka fragment file di relics juga terhapus
![image](https://github.com/v0rein/SISOP-4-2024-MH-IT23/assets/143814923/7725b979-a213-4ee2-8ea3-5f3f96826152)

Cek directory report menggunakan samba share
![image](https://github.com/v0rein/SISOP-4-2024-MH-IT23/assets/143814923/bd3bc3b7-d1c8-4eb4-a36b-4d1e6f019be0)


