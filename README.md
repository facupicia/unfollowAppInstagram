# Instagram Unfollowers Remover

![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![Language](https://img.shields.io/badge/language-JavaScript-yellow)
![Last Updated](https://img.shields.io/badge/last%20updated-Agosto%202025-blue)

Una herramienta para **identificar y dejar de seguir usuarios en Instagram** de forma simple.  
Nació como un conjunto de scripts en JavaScript para ejecutar directamente desde la consola del navegador.  
Hoy podés usar la [versión GUI](https://43t6lx.csb.app/) o los scripts manuales (para usuarios avanzados).

---

## 🧭 Tabla de contenido

- [Actualización importante](#-actualización-importante)
- [Avisos y legalidad](#-avisos-y-legalidad)
- [Requisitos](#-requisitos)
- [Modo consola (scripts)](#-modo-consola-scripts)
- [Funciones disponibles](#-funciones-disponibles)
- [Inicialización de lista](#-inicialización-de-lista)
- [Recomendaciones](#-recomendaciones)
- [Legal y licencia](#-legal-y-licencia)
- [English version below](#-english-version)

---

## 🚀 Actualización importante (Agosto 2025)

Si preferís los scripts originales (por ejemplo, para navegadores que no sean Chrome), podés usarlos sin problema.  
⚠️ **Atención:** estos scripts llaman directamente a la API de Instagram y, aunque son rápidos, tienen más riesgo de detección.  
Por eso, se recomienda la extensión o la versión GUI para la mayoría de los usuarios.

👉 [Abrir herramienta GUI](https://43t6lx.csb.app/)

---

## ⚠️ Avisos y legalidad

- Automatizar acciones en Instagram puede **violar sus Términos de Servicio**.  
- No compartas tus credenciales ni ejecutes código que no entiendas.  
- Este proyecto **no está afiliado, asociado ni respaldado por Instagram**.  
- Usalo **bajo tu propio riesgo**.

---

## 🧩 Requisitos

- Navegador web (Chrome recomendado).
- Tener sesión iniciada en Instagram.
- Saber abrir la consola de desarrollador (DevTools):  
  - `Ctrl + Shift + J` (Windows)  
  - `⌘ + ⌥ + I` (Mac)

---

## 🧠 Modo consola (scripts)

1. Iniciá sesión en tu cuenta de Instagram.  
2. Abrí la consola (`Ctrl+Shift+J` o `⌘+⌥+I`).  
3. Pegá y ejecutá la **primera función**.  
4. Actualizá la página.  
5. Volvé a abrir la consola e inicializá tu lista de usuarios (`listOfUsers`).  
6. Pegá y ejecutá la **segunda función**.

---

## ⚙️ Funciones disponibles

### 1️⃣ `getFollowings()`

Recolecta los usuarios que seguís actualmente en la vista de Instagram.

```javascript
/**
 * getFollowings()
 * Escanea la lista de "siguiendo" visible y devuelve un array de usernames.
 */
function getFollowings() {
  const list = [];
  const nodes = document.querySelectorAll('a[href^="/"][role="link"]');

  nodes.forEach(node => {
    try {
      const href = node.getAttribute('href');
      if (href && /^\/[A-Za-z0-9._]+\/?$/.test(href)) {
        const username = href.replace(/\//g, '');
        if (username && !list.includes(username)) list.push(username);
      }
    } catch (e) {}
  });

  return list;
}

