# Materi Lanjutan - Studi Kasus Pendaftaran Mahasiswa Baru dengan Java Spring Boot, Thymeleaf, dan Bootstrap (Tanpa Database)

---

## Struktur Direktori
Berikut adalah struktur direktori proyek yang akan dibuat:

```
src/main/java/com/example/belajar_spring
    ├── config
    │   └── LoginInterceptor.java
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
### 2.1. Membuat Middleware Login Session
#### LoginInterceptor
```java name=src/main/java/com/example/belajar_spring/config/LoginInterceptor.java
package com.example.belajar_spring.config;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

@Component
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // Allow access to login and registration pages
        String uri = request.getRequestURI();
        if (uri.startsWith("/auth") || uri.equals("/")) {
            return true;
        }

        // Check if user is logged in
        Object userSession = request.getSession().getAttribute("loggedInUser");
        if (userSession == null) {
            response.sendRedirect("/auth/login");
            return false;
        }

        return true;
    }
}
```

#### WebConfig
```java name=src/main/java/com/example/belajar_spring/config/WebConfig.java
package com.example.belajar_spring.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private LoginInterceptor loginInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor).addPathPatterns("/**");
    }
}
```

---
### 2.3. Membuat Service

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

### 2.4. Membuat Controller

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
package com.example.belajar_spring.controller;

import com.example.belajar_spring.model.Jurusan;
import com.example.belajar_spring.service.JurusanService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/jurusan")
public class JurusanController {
    @Autowired
    private JurusanService jurusanService;

    @GetMapping
    public String listJurusan(Model model) {
        model.addAttribute("jurusanList", jurusanService.getAllJurusan());
        return "jurusan/index";
    }

    @GetMapping("/add")
    public String addJurusanForm(Model model) {
        model.addAttribute("jurusan", new Jurusan());
        return "jurusan/add";
    }

    @PostMapping("/add")
    public String saveJurusan(@ModelAttribute Jurusan jurusan) {
        jurusanService.saveJurusan(jurusan);
        return "redirect:/jurusan";
    }
}
```


#### Controller Login dan Register
```java name=src/main/java/com/example/belajar_spring/controller/AuthController.java
package com.example.belajar_spring.controller;

import com.example.belajar_spring.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpSession;

@Controller
@RequestMapping("/auth")
public class AuthController {
    @Autowired
    private UserService userService;

    @GetMapping("/login")
    public String loginForm() {
        return "auth/login";
    }

    @PostMapping("/login")
    public String login(@RequestParam String username, @RequestParam String password, HttpServletRequest request, Model model) {
        if (userService.login(username, password)) {
            HttpSession session = request.getSession();
            session.setAttribute("loggedInUser", username);
            return "redirect:/mahasiswa";
        }
        model.addAttribute("error", "Username atau password salah!");
        return "auth/login";
    }

    @GetMapping("/logout")
    public String logout(HttpServletRequest request) {
        HttpSession session = request.getSession();
        session.invalidate();
        return "redirect:/auth/login";
    }

    @GetMapping("/register")
    public String registerForm() {
        return "auth/register";
    }

    @PostMapping("/register")
    public String register(@RequestParam String username, @RequestParam String password, Model model) {
        if (userService.register(username, password)) {
            return "redirect:/auth/login";
        }
        model.addAttribute("error", "Username sudah ada!");
        return "auth/register";
    }
}
```

---

### 2.5. Membuat Template HTML

#### Template Login (`login.html`)
```html name=src/main/resources/templates/auth/login.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" th:replace="layout/base :: main-content">
<body>
    <h2>Login</h2>
    <form th:action="@{/auth/login}" method="post">
        <div class="form-group">
            <label for="username">Username</label>
            <input id="username" name="username" type="text" class="form-control">
        </div>
        <div class="form-group">
            <label for="password">Password</label>
            <input id="password" name="password" type="password" class="form-control">
        </div>
        <button type="submit" class="btn btn-primary mt-2">Login</button>
        <p class="mt-3">Belum punya akun? <a href="/auth/register">Register</a></p>
        <p th:if="${error}" th:text="${error}" class="text-danger"></p>
    </form>
</body>
</html>
```

#### Template Register (`register.html`)
```html name=src/main/resources/templates/auth/register.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" th:replace="layout/base :: main-content">
<body>
    <h2>Register</h2>
    <form th:action="@{/auth/register}" method="post">
        <div class="form-group">
            <label for="username">Username</label>
            <input id="username" name="username" type="text" class="form-control">
        </div>
        <div class="form-group">
            <label for="password">Password</label>
            <input id="password" name="password" type="password" class="form-control">
        </div>
        <button type="submit" class="btn btn-primary mt-2">Register</button>
        <p th:if="${error}" th:text="${error}" class="text-danger"></p>
    </form>
</body>
</html>
```

#### Layout Base (`base.html`)
```html name=src/main/resources/templates/layout/base.html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CRUD Mahasiswa</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4">
        <div th:replace="layout/header :: header"></div>
        <main>
            <div th:replace="~{::main-content}"></div>
        </main>
        <div th:replace="layout/footer :: footer"></div>
    </div>
</body>
</html>
```

#### Header (`header.html`)
```html name=src/main/resources/templates/layout/header.html
<header>
    <h1 class="text-center">Pendaftaran Mahasiswa Baru</h1>
    <nav>
        <a href="/mahasiswa" class="btn btn-primary">Mahasiswa</a>
        <a href="/jurusan" class="btn btn-secondary">Jurusan</a>
    </nav>
</header>
```

#### Footer (`footer.html`)
```html name=src/main/resources/templates/layout/footer.html
<footer class="text-center mt-4">
    <p>&copy; 2025 Pendaftaran Mahasiswa Baru. All Rights Reserved.</p>
</footer>
```

#### Daftar Mahasiswa (`index.html`)
```html name=src/main/resources/templates/mahasiswa/index.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" th:replace="layout/base :: main-content">
<body>
    <h2>Daftar Mahasiswa</h2>
    <a href="/mahasiswa/add" class="btn btn-success">Tambah Mahasiswa</a>
    <table class="table mt-3">
        <thead>
            <tr>
                <th>Nama</th>
                <th>Email</th>
                <th>Jurusan</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
        <tr th:each="mahasiswa : ${mahasiswaList}">
            <td th:text="${mahasiswa.nama}"></td>
            <td th:text="${mahasiswa.email}"></td>
            <td th:text="${mahasiswa.jurusan}"></td>
            <td>
                <a th:href="@{/mahasiswa/delete/{id}(id=${mahasiswa.id})}" class="btn btn-danger">Hapus</a>
            </td>
        </tr>
        </tbody>
    </table>
</body>
</html>
```

#### Form Tambah Mahasiswa (`add.html`)
```html name=src/main/resources/templates/mahasiswa/add.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" th:replace="layout/base :: main-content">
<body>
    <h2>Tambah Mahasiswa</h2>
    <form th:action="@{/mahasiswa/add}" method="post">
        <div class="form-group">
            <label for="nama">Nama</label>
            <input id="nama" name="nama" type="text" class="form-control">
        </div>
        <div class="form-group">
            <label for="email">Email</label>
            <input id="email" name="email" type="email" class="form-control">
        </div>
        <div class="form-group">
            <label for="jurusan">Jurusan</label>
            <select id="jurusan" name="jurusan" class="form-control">
                <option th:each="jurusan : ${jurusanList}" th:value="${jurusan.namaJurusan}" th:text="${jurusan.namaJurusan}"></option>
            </select>
        </div>
        <button type="submit" class="btn btn-primary mt-2">Simpan</button>
    </form>
</body>
</html>
```

---

## 3. Menjalankan Aplikasi
1. Jalankan aplikasi dengan:
   ```bash
   mvn spring-boot:run
   ```
2. Akses di `http://localhost:8080`.

---

## 4. Kesimpulan
Dengan tutorial ini, Anda telah berhasil membuat aplikasi CRUD Mahasiswa Baru menggunakan Java Spring Boot, Thymeleaf, dan Bootstrap tanpa menggunakan database.
