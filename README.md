# streamig-go
Proyecto de Sistema de streaming en Go.
# Main.go
package main

import (
	"errors"
	"fmt"
)

// Estructura para representar un Usuario
type Usuario struct {
	nombre string
	email  string
	activo bool
}

// Método setter para el nombre del usuario
func (u *Usuario) SetNombre(nombre string) {
	u.nombre = nombre
}

// Método getter para el nombre del usuario
func (u *Usuario) GetNombre() string {
	return u.nombre
}

// Método para activar el usuario
func (u *Usuario) Activar() {
	u.activo = true
}

// Método para desactivar el usuario
func (u *Usuario) Desactivar() {
	u.activo = false
}

// Estructura para representar Contenido
type Contenido struct {
	titulo      string
	descripcion string
	disponible  bool
}

// Método para marcar contenido como disponible
func (c *Contenido) MarcarComoDisponible() {
	c.disponible = true
}

// Método para marcar contenido como no disponible
func (c *Contenido) MarcarComoNoDisponible() {
	c.disponible = false
}

// Estructura para representar la Plataforma de Streaming
type Plataforma struct {
	usuarios  map[string]*Usuario
	contenido map[string]*Contenido
}

// Método para agregar un usuario a la plataforma
func (p *Plataforma) AgregarUsuario(email string, nombre string) error {
	if _, existe := p.usuarios[email]; existe {
		return errors.New("el usuario ya existe")
	}
	p.usuarios[email] = &Usuario{nombre: nombre, email: email, activo: false}
	return nil
}

// Método para agregar contenido a la plataforma
func (p *Plataforma) AgregarContenido(titulo string, descripcion string) {
	p.contenido[titulo] = &Contenido{titulo: titulo, descripcion: descripcion, disponible: false}
}

// Método para activar un usuario
func (p *Plataforma) ActivarUsuario(email string) error {
	if usuario, existe := p.usuarios[email]; existe {
		usuario.Activar()
		return nil
	}
	return errors.New("usuario no encontrado")
}

// Método para mostrar todos los usuarios
func (p *Plataforma) MostrarUsuarios() {
	for _, usuario := range p.usuarios {
		fmt.Printf("Nombre: %s, Email: %s, Activo: %t\n", usuario.GetNombre(), usuario.email, usuario.activo)
	}
}

func main() {
	plataforma := Plataforma{
		usuarios:  make(map[string]*Usuario),
		contenido: make(map[string]*Contenido),
	}

	// Agregar usuarios
	err := plataforma.AgregarUsuario("samuel@uide.com", "Usuario Uno")
	if err != nil {
		fmt.Println(err)
	}

	// Activar usuario
	err = plataforma.ActivarUsuario("samuel@uide.com")
	if err != nil {
		fmt.Println(err)
	}

	// Mostrar usuarios
	plataforma.MostrarUsuarios()

	// Agregar contenido
	plataforma.AgregarContenido("Película 1", "Descripción de la película 1")
}

# Models / contenido.go
package models

import (
	"fmt"
)

type Usuario struct {
	nombre string
	email  string
	edad   int
}

func (u *Usuario) SetNombre(nombre string) {
	u.nombre = nombre
}

func (u *Usuario) GetNombre() string {
	return u.nombre
}

func (u *Usuario) SetEmail(email string) error {
	if len(email) < 5 {
		return fmt.Errorf("email inválido")
	}
	u.email = email
	return nil
}

func (u *Usuario) GetEmail() string {
	return u.email
}

func (u *Usuario) SetEdad(edad int) error {
	if edad < 0 {
		return fmt.Errorf("la edad no puede ser negativa")
	}
	u.edad = edad
	return nil
}

func (u *Usuario) GetEdad() int {
	return u.edad
}

# Models / usuario.go
package models

import (
	"fmt"
)

type Usuario struct {
	nombre string
	email  string
	edad   int
}

func (u *Usuario) SetNombre(nombre string) {
	u.nombre = nombre
}

func (u *Usuario) GetNombre() string {
	return u.nombre
}

func (u *Usuario) SetEmail(email string) error {
	if len(email) < 5 {
		return fmt.Errorf("email inválido")
	}
	u.email = email
	return nil
}

func (u *Usuario) GetEmail() string {
	return u.email
}

func (u *Usuario) SetEdad(edad int) error {
	if edad < 0 {
		return fmt.Errorf("la edad no puede ser negativa")
	}
	u.edad = edad
	return nil
}

func (u *Usuario) GetEdad() int {
	return u.edad
}

# Interfaces / reproducible
package interfaces

type Reproducible interface {
	Reproducir() string
}

# go.mod
module sistema-streaming

go 1.24.4


