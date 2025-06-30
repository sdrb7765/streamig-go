# db 
# connect.go
package db

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

var DB *sql.DB

func ConectarDB() {
	var err error
	DB, err = sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/streaming")
	if err != nil {
		panic("Error de conexión: " + err.Error())
	}

	err = DB.Ping()
	if err != nil {
		panic("No se puede hacer ping a la base de datos")
	}

	fmt.Println("Conexión exitosa a la base de datos")
}
# Handler
# api_peliculas_concurrente.go
package db

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

var DB *sql.DB

func ConectarDB() {
	var err error
	DB, err = sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/streaming")
	if err != nil {
		panic("Error de conexión: " + err.Error())
	}

	err = DB.Ping()
	if err != nil {
		panic("No se puede hacer ping a la base de datos")
	}

	fmt.Println("Conexión exitosa a la base de datos")
}
# api_peliculas.go
package handlers

import (
	"encoding/json"
	"html/template"
	"net/http"
	"streaming/models"
)

func ObtenerPeliculas(w http.ResponseWriter, r *http.Request) {
	peliculas, err := models.ObtenerPeliculas()
	if err != nil {
		http.Error(w, "Error al obtener películas", http.StatusInternalServerError)
		return
	}

	if r.Header.Get("Content-Type") == "application/json" {
		json.NewEncoder(w).Encode(peliculas)
		return
	}

	tmpl, _ := template.ParseFiles("templates/base.html", "templates/peliculas.html")
	tmpl.Execute(w, peliculas)
}

func CrearPelicula(w http.ResponseWriter, r *http.Request) {
	if r.Method == http.MethodPost {
		titulo := r.FormValue("titulo")
		genero := r.FormValue("genero")
		modalidad := r.FormValue("modalidad")
		err := models.CrearPelicula(titulo, genero, modalidad)
		if err != nil {
			http.Error(w, "Error al guardar película", http.StatusInternalServerError)
			return
		}
		http.Redirect(w, r, "/peliculas", http.StatusSeeOther)
	}
}
# home.go
package handlers

import (
	"html/template"
	"net/http"
)

func Home(w http.ResponseWriter, r *http.Request) {
	tmpl, _ := template.ParseFiles("templates/base.html", "templates/index.html")
	tmpl.Execute(w, nil)
}
# models
# interfaces.go
package models

type Contenido interface {
	GetTitulo() string
	GetModalidad() string
}
# pelicula.go
package models

import (
	"fmt"
	"streaming/db"
)

type Pelicula struct {
	ID        int
	Titulo    string
	Genero    string
	Modalidad string
}

// Encapsulamiento (setter)
func (p *Pelicula) SetTitulo(titulo string) error {
	if len(titulo) < 2 {
		return fmt.Errorf("el título es muy corto")
	}
	p.Titulo = titulo
	return nil
}

// Métodos para interfaz
func (p Pelicula) GetTitulo() string {
	return p.Titulo
}

func (p Pelicula) GetModalidad() string {
	return p.Modalidad
}

// Crear película
func CrearPelicula(titulo, genero, modalidad string) error {
	p := Pelicula{}
	if err := p.SetTitulo(titulo); err != nil {
		return err
	}
	p.Genero = genero
	p.Modalidad = modalidad

	_, err := db.DB.Exec("INSERT INTO peliculas (titulo, genero, modalidad) VALUES (?, ?, ?)", p.Titulo, p.Genero, p.Modalidad)
	return err
}

// Obtener películas
func ObtenerPeliculas() ([]Pelicula, error) {
	rows, err := db.DB.Query("SELECT * FROM peliculas")
	if err != nil {
		return nil, err
	}
	defer rows.Close()

	var peliculas []Pelicula
	for rows.Next() {
		var p Pelicula
		rows.Scan(&p.ID, &p.Titulo, &p.Genero, &p.Modalidad)
		peliculas = append(peliculas, p)
	}
	return peliculas, nil
}
# templates
# base.html 
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>{{ .Titulo }}</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #111;
            color: white;
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #e50914;
            padding: 10px 20px;
            text-align: center;
        }
        header h1 {
            margin: 0;
        }
        nav a {
            margin: 0 10px;
            color: white;
            text-decoration: none;
        }
        .contenedor {
            padding: 20px;
        }
        .tarjeta {
            background-color: #222;
            border-radius: 5px;
            padding: 10px;
            margin: 10px;
            width: 220px;
            display: inline-block;
            vertical-align: top;
        }
        .tarjeta img {
            width: 100%;
            height: auto;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <header>
        <h1>StreamingGo</h1>
        <nav>
            <a href="/">Inicio</a>
            <a href="/crear">Agregar Película</a>
        </nav>
    </header>

    <div class="contenedor">
        {{ template "contenido" . }}
    </div>
</body>
</html>

# crear.html
{{ define "contenido" }}
    <h2>Agregar Nueva Película</h2>
    <form method="POST" action="/crear">
        <label for="titulo">Título:</label><br>
        <input type="text" name="titulo" required><br><br>

        <label for="director">Director:</label><br>
        <input type="text" name="director" required><br><br>

        <label for="genero">Género:</label><br>
        <input type="text" name="genero" required><br><br>

        <label for="duracion">Duración (min):</label><br>
        <input type="number" name="duracion" required><br><br>

        <label for="imagen">URL de imagen:</label><br>
        <input type="text" name="imagen" required><br><br>

        <input type="submit" value="Guardar Película">
    </form>
{{ end }}

# index.html
{{ define "contenido" }}
    <h2>Películas Disponibles</h2>
    {{ range .Peliculas }}
        <div class="tarjeta">
            <img src="{{ .ImagenURL }}" alt="{{ .Titulo }}">
            <h3>{{ .Titulo }}</h3>
            <p><strong>Director:</strong> {{ .Director }}</p>
            <p><strong>Género:</strong> {{ .Genero }}</p>
            <p><strong>Duración:</strong> {{ .Duracion }} min</p>
        </div>
    {{ else }}
        <p>No hay películas disponibles.</p>
    {{ end }}
{{ end }}

# peliculas.html
{{define "contenido"}}
<h1>Películas registradas</h1>
<ul>
    {{range .}}
    <li>{{.Titulo}} - {{.Genero}} - {{.Modalidad}}</li>
    {{end}}
</ul>
<h2>Agregar nueva película</h2>
<form action="/peliculas/crear" method="POST">
    <input name="titulo" placeholder="Título"><br>
    <input name="genero" placeholder="Género"><br>
    <input name="modalidad" placeholder="Modalidad"><br>
    <button type="submit">Guardar</button>
</form>
{{end}}

# handler_test.go
package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestHomeHandler(t *testing.T) {
	req, _ := http.NewRequest("GET", "/", nil)
	res := httptest.NewRecorder()

	handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Inicio"))
	})
	handler.ServeHTTP(res, req)

	if res.Code != 200 {
		t.Errorf("esperado código 200, obtuve %d", res.Code)
	}
}

# main.go
package main

import (
	"fmt"
	"net/http"
	"streaming/db"
)

func main() {
	db.ConectarDB()
	CargarRutas()
	fmt.Println("Servidor iniciado en http://localhost:8080")
	http.ListenAndServe(":8080", nil)
}

# routes.go
package main

import (
	"net/http"
	"streaming/handlers"
)

func CargarRutas() {
	http.HandleFunc("/", handlers.Home)
	http.HandleFunc("/peliculas", handlers.ObtenerPeliculas)
	http.HandleFunc("/peliculas/crear", handlers.CrearPelicula)
	http.HandleFunc("/api/peliculas/concurrente", handlers.CargarPeliculasConGoroutine)

	http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("static"))))
}

# go.mod
module streaming

go 1.24.4

# go.sun
filippo.io/edwards25519 v1.1.0 h1:FNf4tywRC1HmFuKW5xopWpigGjJKiJSV0Cqo0cJWDaA=
filippo.io/edwards25519 v1.1.0/go.mod h1:BxyFTGdWcka3PhytdK4V28tE5sGfRvvvRV7EaN4VDT4=
github.com/go-sql-driver/mysql v1.9.3 h1:U/N249h2WzJ3Ukj8SowVFjdtZKfu9vlLZxjPXV1aweo=
github.com/go-sql-driver/mysql v1.9.3/go.mod h1:qn46aNg1333BRMNU69Lq93t8du/dwxI64Gl8i5p1WMU=

# SQL
CREATE DATABASE streaming;
USE streaming;

CREATE TABLE peliculas (
  id INT AUTO_INCREMENT PRIMARY KEY,
  titulo VARCHAR(100),
  director VARCHAR(100),
  genero VARCHAR(50),
  duracion INT,
  imagen_url VARCHAR(255),
  disponible BOOLEAN DEFAULT TRUE
);

CREATE TABLE series (
  id INT AUTO_INCREMENT PRIMARY KEY,
  titulo VARCHAR(100),
  temporadas INT,
  genero VARCHAR(50),
  activa BOOLEAN DEFAULT TRUE
);


 
