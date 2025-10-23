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

### 1️⃣ `startScript()`

Recolecta los usuarios que seguís actualmente en la vista de Instagram.

```javascript
/**
 * startScript()
 * Escanea la lista de "siguiendo" visible y devuelve un array de usernames.
 */

function getCookie(name) {
  const cookies = `; ${document.cookie}`;
  const parts = cookies.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

function afterUrlGenerator(after) {
  return `https://www.instagram.com/graphql/query/?query_hash=3dec7e2c57367ef3da3d987d89f9dbc8&variables={"id":"${ds_user_id}","include_reel":"true","fetch_mutual":"false","first":"24","after":"${after}"}`;
}

let followedPeople,
  csrftoken = getCookie("csrftoken"),
  ds_user_id = getCookie("ds_user_id"),
  initialURL = `https://www.instagram.com/graphql/query/?query_hash=3dec7e2c57367ef3da3d987d89f9dbc8&variables={"id":"${ds_user_id}","include_reel":"true","fetch_mutual":"false","first":"24"}`,
  doNext = true,
  followingList = [],
  fetchedCount = 0,
  scrollCycle = 0;

async function startScript() {
  console.log("%c 🟢 Starting... please wait", "background: #222; color: #bada55; font-size: 20px;");

  while (doNext) {
    let res;
    try {
      res = await fetch(initialURL).then(r => r.json());
    } catch (err) {
      console.log("Fetch error, retrying...");
      continue;
    }

    followedPeople ||= res.data.user.edge_follow.count;
    doNext = res.data.user.edge_follow.page_info.has_next_page;
    initialURL = afterUrlGenerator(res.data.user.edge_follow.page_info.end_cursor);

    const edges = res.data.user.edge_follow.edges;
    fetchedCount += edges.length;

    // 👉 Ahora guardamos todos (sin filtro)
    edges.forEach(e => followingList.push(e.node));

    console.clear();
    console.log(
      `%c Progress ${fetchedCount}/${followedPeople} (${parseInt(
        100 * (fetchedCount / followedPeople)
      )}%)`,
      "background: #222; color: #bada55; font-size: 30px;"
    );
    console.log("%c Still fetching... be patient!", "background: #222; color: #FC4119; font-size: 14px;");

    await sleep(Math.floor(400 * Math.random()) + 1000);

    scrollCycle++;
    if (scrollCycle > 6) {
      scrollCycle = 0;
      console.log(
        "%c Sleeping 10 seconds to prevent temporary block...",
        "background: #222; color: #FF0000; font-size: 20px;"
      );
      await sleep(10000);
    }
  }

  // Guardar archivo
  const jsonContent = JSON.stringify(followingList, null, 2);
  const fileName = "followingList.json";
  const blob = new Blob([jsonContent], { type: "application/json" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = fileName;
  a.click();

  console.log("%c ✅ DONE! Archivo descargado: followingList.json", "background: #222; color: #bada55; font-size: 20px;");
}

startScript();
```

👉 [Abrir herramienta GUI](https://43t6lx.csb.app/) 
- 👉Pegar el archivo que dejo el primer script
- 👉Refrescar la pagina del Instagram 
- 👉Crear esta variable 
```javascript
const listOfUsers = //PASTE HERE THE LIST OF USERS FROM YOUR CLIPBOARD, RESULTS FROM GUI TOOL
```

### 1️⃣ `startUnfollow()`

Recolecta los usuarios que seguís actualmente en la vista de Instagram.

```javascript
/**
 * startUnfollow()
 * corre el script con la lista ya cargada!!
 */
function getCookie(name) {
  const cookies = `; ${document.cookie}`;
  const parts = cookies.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
  return null;
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

function unfollowUserUrl(id) {
  return `https://www.instagram.com/web/friendships/${id}/unfollow/`;
}

const csrftoken = getCookie('csrftoken');

const startUnfollow = async () => {
  let total = 0;
  let batchCount = 0;

  for (const user of listOfUsers) {
    try {
      await fetch(unfollowUserUrl(user.id), {
        headers: {
          'content-type': 'application/x-www-form-urlencoded',
          'x-csrftoken': csrftoken,
        },
        method: 'POST',
        mode: 'cors',
        credentials: 'include',
      });
      console.log(`Unfollowed ${++total}/${listOfUsers.length}`);
    } catch (e) {
      console.error('Error al hacer unfollow:', e);
    }

    await sleep(Math.floor(2000 * Math.random()) + 4000);

    if (++batchCount >= 15) {
      console.log(
        '%cDurmiendo 1.5 minutos para evitar bloqueos temporales...',
        'background: #222; color: #FF0000; font-size: 18px;'
      );
      batchCount = 0;
      await sleep(100000); // 1.5 minutos
    }
  }

  console.log(
    '%c¡Listo! Terminó de hacer unfollow a todos.',
    'background: #222; color: #bada55; font-size: 20px;'
  );
};

startUnfollow();

```



