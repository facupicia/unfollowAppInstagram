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
  scrollCycle = 0,
  MAX_TO_SAVE = 300; // 👈 límite de followers

async function startScript() {
  console.log("%c 🟢 Starting... please wait", "background: #222; color: #bada55; font-size: 20px;");

  while (doNext && followingList.length < MAX_TO_SAVE) {
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

    // Solo agregamos hasta llegar a 300
    for (const e of edges) {
      if (followingList.length >= MAX_TO_SAVE) {
        doNext = false;
        break;
      }
      followingList.push(e.node);
      fetchedCount++;
    }

    console.clear();
    console.log(
      `%c Progress ${followingList.length}/${Math.min(followedPeople, MAX_TO_SAVE)} (${parseInt(
        100 * (followingList.length / MAX_TO_SAVE)
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
  const fileName = "followingList_300.json";
  const blob = new Blob([jsonContent], { type: "application/json" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = fileName;
  a.click();

  console.log(`%c ✅ DONE! Archivo con ${followingList.length} usuarios guardado: followingList_300.json`, "background: #222; color: #bada55; font-size: 20px;");
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

> **v2 — bloqueos suaves detectados.** Instagram devuelve `200 OK` con `status: "fail"` cuando empieza a aplicar rate limit, por eso el script "sigue corriendo" pero no descuenta. Esta versión **verifica la respuesta**, reintenta y aplica pausas más largas (3-5 min entre lotes + 10 min si detecta patrón de bloqueo).

```javascript
/**
 * startUnfollow() — v2
 * Verifica cada respuesta, reintenta y aplica backoff ante bloqueos suaves.
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

// ---- Configuración ----
const BATCH_SIZE         = 10;     // unfollows por lote
const BATCH_COOLDOWN_MIN = 180000; // 3 min
const BATCH_COOLDOWN_MAX = 300000; // 5 min
const ACTION_DELAY_MIN   = 6000;   // 6 s mínimo entre unfollows
const ACTION_DELAY_MAX   = 14000;  // 14 s máximo
const FAILURE_THRESHOLD  = 3;      // fallos seguidos → pausa larga
const LONG_PAUSE         = 600000; // 10 min al detectar bloqueo
const MAX_RETRIES        = 2;      // reintentos por usuario

const startUnfollow = async () => {
  let total = 0;
  let okCount = 0;
  let failCount = 0;
  let batchCount = 0;
  let consecutiveFailures = 0;

  for (const user of listOfUsers) {
    let attempt = 0;
    let success = false;

    while (attempt < MAX_RETRIES && !success) {
      try {
        const res = await fetch(unfollowUserUrl(user.id), {
          headers: {
            'content-type': 'application/x-www-form-urlencoded',
            'x-csrftoken': csrftoken,
            'x-ig-app-id': '936619743392459',
            'x-requested-with': 'XMLHttpRequest',
            'referer': 'https://www.instagram.com/',
            'origin': 'https://www.instagram.com',
          },
          method: 'POST',
          mode: 'cors',
          credentials: 'include',
        });

        if (res.ok) {
          const data = await res.json().catch(() => ({}));
          if (data.status === 'ok') {
            success = true;
            okCount++;
            consecutiveFailures = 0;
          } else {
            console.warn(`%c[!] ${user.username} → status=${data.status || 'desconocido'}`, 'color: orange');
            consecutiveFailures++;
          }
        } else if (res.status === 429) {
          console.warn('%c[!] HTTP 429 (rate limit). Pausando 5 min...', 'color: orange');
          await sleep(300000);
          consecutiveFailures++;
        } else {
          console.warn(`%c[!] ${user.username} → HTTP ${res.status}`, 'color: orange');
          consecutiveFailures++;
        }
      } catch (e) {
        console.error(`%c[x] ${user.username} → ${e.message}`, 'color: red');
        consecutiveFailures++;
      }
      attempt++;
      if (!success && attempt < MAX_RETRIES) await sleep(3000);
    }

    if (!success) failCount++;
    total++;

    console.log(
      `%c[${total}/${listOfUsers.length}] ${user.username} → ${success ? 'OK' : 'FALLÓ'} | ok: ${okCount} | fallidos: ${failCount}`,
      `color: ${success ? '#bada55' : '#FC4119'}; font-weight: bold`
    );

    // Pausa larga si hay muchos fallos consecutivos
    if (consecutiveFailures >= FAILURE_THRESHOLD) {
      console.log(
        `%c⏸ ${consecutiveFailures} fallos seguidos. Pausa de 10 min para que Instagram se calme...`,
        'background: #222; color: #FF0000; font-size: 18px;'
      );
      consecutiveFailures = 0;
      await sleep(LONG_PAUSE);
    } else {
      // Delay aleatorio entre acciones (6-14 s)
      await sleep(
        ACTION_DELAY_MIN + Math.floor(Math.random() * (ACTION_DELAY_MAX - ACTION_DELAY_MIN))
      );
    }

    // Cooldown entre lotes
    if (++batchCount >= BATCH_SIZE) {
      batchCount = 0;
      const cooldown = BATCH_COOLDOWN_MIN +
        Math.floor(Math.random() * (BATCH_COOLDOWN_MAX - BATCH_COOLDOWN_MIN));
      console.log(
        `%c☕ Lote completado. Cooldown de ${Math.round(cooldown / 60000)} min...`,
        'background: #222; color: #FFD700; font-size: 18px;'
      );
      await sleep(cooldown);
    }
  }

  console.log(
    `%c✅ Terminado. ${okCount} unfollows OK · ${failCount} fallidos.`,
    'background: #222; color: #bada55; font-size: 20px;'
  );
};

startUnfollow();
```




