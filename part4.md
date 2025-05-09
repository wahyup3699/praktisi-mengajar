# Materi Lanjutan - Studi Kasus Pendaftaran Mahasiswa Baru dengan Java Spring Boot, Thymeleaf, dan Bootstrap (Tanpa Database)

---

## Struktur Direktori
Berikut adalah struktur direktori proyek yang akan dibuat:

```
src/main/java/com/example/belajar_spring
    ├── controller
    ├── model
    ├── service
    └── BelajarSpringApplication.java
src/main/resources/
    ├── templates/
    │   ├── layout/
    │   │   ├── base.html
    │   │   ├── header.html
    │   │   └── footer.html
    │   ├── mahasiswa/
    │   │   ├── index.html
    │   │   └── add.html
    │   ├── jurusan/
    │   │   ├── index.html
    │   │   └── add.html
    │   └── auth/
    │       ├── login.html
    │       └── register.html
    ├── static/
    │   ├── css/
    │   └── js/
    └── application.properties
```

---

## 2. Implementasi

### 2.1. Membuat Model
Model adalah representasi dari entitas data.

#### Model Mahasiswa
```java name=src/main/java/com/example/belajar_spring/model/Mahasiswa.java
package com.example.belajar_spring.model;

public class Mahasiswa {
    private Long id;
    private String nama;
    private String email;
    private String jurusan;

    public Mahasiswa() {}

    public Mahasiswa(Long id, String nama, String email, String jurusan) {
        this.id = id;
        this.nama = nama;
        this.email = email;
        this.jurusan = jurusan;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNama() {
        return nama;
    }

    public void setNama(String nama) {
        this.nama = nama;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getJurusan() {
        return jurusan;
    }

    public void setJurusan(String jurusan) {
        this.jurusan = jurusan;
    }
}
```

#### Model Jurusan
```java name=src/main/java/com/example/belajar_spring/model/Jurusan.java
package com.example.belajar_spring.model;

public class Jurusan {
    private Long id;
    private String namaJurusan;

    public Jurusan() {}

    public Jurusan(Long id, String namaJurusan) {
        this.id = id;
        this.namaJurusan = namaJurusan;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNamaJurusan() {
        return namaJurusan;
    }

    public void setNamaJurusan(String namaJurusan) {
        this.namaJurusan = namaJurusan;
    }
}
```

#### Model Pengguna
```java name=src/main/java/com/example/belajar_spring/model/User.java
package com.example.belajar_spring.model;

public class User {
    private String username;
    private String password;

    public User() {}

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

---

### 2.2. Membuat Service

#### Service Mahasiswa
```java name=src/main/java/com/example/belajar_spring/service/MahasiswaService.java
package com.example.belajar_spring.service;

import com.example.belajar_spring.model.Mahasiswa;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class MahasiswaService {
    private final List<Mahasiswa> mahasiswaList = new ArrayList<>();
    private Long idCounter = 1L;

    public List<Mahasiswa> getAllMahasiswa() {
        return mahasiswaList;
    }

    public Mahasiswa saveMahasiswa(Mahasiswa mahasiswa) {
        if (mahasiswa.getId() == null) {
            mahasiswa.setId(idCounter++);
            mahasiswaList.add(mahasiswa);
        } else {
            deleteMahasiswa(mahasiswa.getId());
            mahasiswaList.add(mahasiswa);
        }
        return mahasiswa;
    }

    public void deleteMahasiswa(Long id) {
        mahasiswaList.removeIf(mahasiswa -> mahasiswa.getId().equals(id));
    }

    public Optional<Mahasiswa> findMahasiswaById(Long id) {
        return mahasiswaList.stream().filter(m -> m.getId().equals(id)).findFirst();
    }
}
```

#### Service Jurusan
```java name=src/main/java/com/example/belajar_spring/service/JurusanService.java
package com.example.belajar_spring.service;

import com.example.belajar_spring.model.Jurusan;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

@Service
public class JurusanService {
    private final List<Jurusan> jurusanList = new ArrayList<>();
    private Long idCounter = 1L;

    public List<Jurusan> getAllJurusan() {
        return jurusanList;
    }

    public Jurusan saveJurusan(Jurusan jurusan) {
        if (jurusan.getId() == null) {
            jurusan.setId(idCounter++);
            jurusanList.add(jurusan);
        }
        return jurusan;
    }
}
```

#### Service Pengguna
```java name=src/main/java/com/example/belajar_spring/service/UserService.java
package com.example.belajar_spring.service;

import com.example.belajar_spring.model.User;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class UserService {
    private final Map<String, String> users = new HashMap<>();

    public boolean register(String username, String password) {
        if (users.containsKey(username)) {
            return false; // Username already exists
        }
        users.put(username, password);
        return true;
    }

    public boolean login(String username, String password) {
        return users.containsKey(username) && users.get(username).equals(password);
    }
}
```

---

### 2.3. Membuat Controller

#### Controller Mahasiswa
```java name=src/main/java/com/example/belajar_spring/controller/MahasiswaController.java
package com.example.belajar_spring.controller;

import com.example.belajar_spring.model.Mahasiswa;
import com.example.belajar_spring.service.JurusanService;
import com.example.belajar_spring.service.MahasiswaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/mahasiswa")
public class MahasiswaController {
    @Autowired
    private MahasiswaService mahasiswaService;

    @Autowired
    private JurusanService jurusanService;

    @GetMapping
    public String listMahasiswa(Model model) {
        model.addAttribute("mahasiswaList", mahasiswaService.getAllMahasiswa());
        return "mahasiswa/index";
    }

    @GetMapping("/add")
    public String addMahasiswaForm(Model model) {
        model.addAttribute("mahasiswa", new Mahasiswa());
        model.addAttribute("jurusanList", jurusanService.getAllJurusan());
        return "mahasiswa/add";
    }

    @PostMapping("/add")
    public String saveMahasiswa(@ModelAttribute Mahasiswa mahasiswa) {
        mahasiswaService.saveMahasiswa(mahasiswa);
        return "redirect:/mahasiswa";
    }

    @GetMapping("/delete/{id}")
    public String deleteMahasiswa(@PathVariable Long id) {
        mahasiswaService.deleteMahasiswa(id);
        return "redirect:/mahasiswa";
    }
}
```

#### Controller Jurusan
```java name=src/main/java/com/example/belajar_spring/controller/JurusanController.java
// (isi seperti pada contoh sebelumnya)
```

#### Controller Login dan Register
```java name=src/main/java/com/example/belajar_spring/controller/AuthController.java
// (isi seperti pada contoh sebelumnya)
```

---

### 2.4. Membuat Template HTML

#### Template Login (`login.html`)
```html name=src/main/resources/templates/auth/login.html
// (isi seperti pada contoh sebelumnya)
```

#### Template Register (`register.html`)
```html name=src/main/resources/templates/auth/register.html
// (isi seperti pada contoh sebelumnya)
```

#### Layout Base (`base.html`)
```html name=src/main/resources/templates/layout/base.html
// (isi seperti pada contoh sebelumnya)
```

#### Header (`header.html`)
```html name=src/main/resources/templates/layout/header.html
// (isi seperti pada contoh sebelumnya)
```

#### Footer (`footer.html`)
```html name=src/main/resources/templates/layout/footer.html
// (isi seperti pada contoh sebelumnya)
```

#### Daftar Mahasiswa (`index.html`)
```html name=src/main/resources/templates/mahasiswa/index.html
// (isi seperti pada contoh sebelumnya)
```

#### Form Tambah Mahasiswa (`add.html`)
```html name=src/main/resources/templates/mahasiswa/add.html
// (isi seperti pada contoh sebelumnya)
```

---

## 3. Menjalankan Aplikasi
1. Jalankan aplikasi dengan:
   ```bash
   mvn spring-boot:run
   ```
2. Akses fitur login di `http://localhost:8080/auth/login`, registrasi di `http://localhost:8080/auth/register`, dan daftar mahasiswa di `http://localhost:8080/mahasiswa`.

---

## 4. Kesimpulan
Dengan tutorial ini, Anda telah berhasil membuat aplikasi CRUD Mahasiswa Baru, fitur login, dan register menggunakan Java Spring Boot, Thymeleaf, dan Bootstrap tanpa menggunakan database.
