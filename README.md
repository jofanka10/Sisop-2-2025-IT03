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

## a. Download

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

## b. Filter
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

## c. Combine
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

## d. Decode
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

Langkahnya cukup singkat. Langkahnya:

- Fungsi membuka file ```Combine.txt```.
- Setelah itu, dilakukan ROT13 dan hasilnya dimasukkan ke file ```Decoded.txt```.
## e. int main()

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

  <h2 id="soal3">Soal3</h2>

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

```c
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
