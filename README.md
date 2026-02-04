# 📝 Guia Ràpida: De Vue a Executable Electron

**Objectiu:** Agafar una web Vue i convertir-la en App Linux (.deb) i App Windows Portable (.zip).

## 1. PREPARAR EL FRONTEND (Vue/Vite)

Abans de tocar res d'Electron, has de preparar la web perquè funcioni sense servidor.

### Edita `vite.config.js` (o `.mjs`)
Afegeix `base: './'` al principi. Això evita la pantalla blanca.

```javascript
export default defineConfig({
  base: './', // <--- CRÍTIC: Rutes relatives
  plugins: [vue(), ...],
  // ...
})
```

### Edita el Router (`src/router/index.js`)
Canvia el mode històric a Hash. Si no, al recarregar o navegar en Electron, fallarà.

```javascript
// Canvia l'import
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  // USA AQUEST MODE:
  history: createWebHashHistory(), 
  routes,
})
```

### Genera el dist

```bash
npm run build
```

Comprova que s'ha creat la carpeta `dist` dins del frontend.

## 2. INICIALITZAR EL PROJECTE ELECTRON

Fes-ho en una carpeta nova separada, al mateix nivell que el frontend.

### Crea l'estructura

```bash
mkdir electron-app
cd electron-app
npm init -y
```

### Instal·la dependències

```bash
npm install electron --save-dev
```

### Configura Electron Forge (L'empaquetador)

```bash
npm install --save-dev @electron-forge/cli
npx electron-forge import
```

(Diga-li que sí a tot).

### Copia el Web
Agafa la carpeta `dist` del pas 1 (frontend) i copia-la dins de la carpeta `electron-app`. Estructura final:

```text
/electron-app
  ├── dist/         <-- La teva web compilada
  ├── node_modules/
  ├── src/          (Ignora la carpeta src que crea el Forge, farem servir main.js)
  ├── package.json
  ├── forge.config.js
  └── main.js       <-- El creem ara
```

## 3. CONFIGURAR ELECTRON (`main.js` i `package.json`)

### Crea el fitxer `main.js` a l'arrel de `electron-app`
Aquest codi evita errors de CORS i carrega el fitxer local.

```javascript
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const win = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false,
      webSecurity: false // <--- CRÍTIC: Evita errors de CORS/Network
    }
  });

  // Carregar el dist/index.html
  win.loadFile(path.join(__dirname, 'dist', 'index.html'));
  
  // Opcional: Treure menú superior
  win.setMenuBarVisibility(false);
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

### Edita el `package.json` (MOLT IMPORTANT PER EVITAR ERRORS)
Assegura't de tenir aquests camps exactament així:

```json
{
  "name": "el-meu-projecte",      
  "productName": "el-meu-projecte", 
  // ^^^ ATTENCIÓ: Tot minúscules i sense espais per evitar error a Linux
  "version": "1.0.0",
  "description": "Projecte examen",
  "author": "Jo",
  "main": "main.js",  // <--- Assegura't que apunta al teu main.js
  // ... scripts i dependencies ...
}
```

### Edita el `forge.config.js`
Per evitar errors de dependències (RPM/Wine) que no tens a l'institut.

```javascript
module.exports = {
  packagerConfig: {
    asar: true,
  },
  rebuildConfig: {},
  makers: [
    {
      name: '@electron-forge/maker-squirrel',
      config: {},
    },
    {
      name: '@electron-forge/maker-zip',
      // AFEGIR 'win32' AQUÍ PER FER EL PORTABLE DE WINDOWS:
      platforms: ['darwin', 'win32', 'linux'],
    },
    {
      name: '@electron-forge/maker-deb',
      config: {},
    },
    // ESBORRA EL BLOC DE 'maker-rpm' SI EXISTEIX PER EVITAR ERRORS
  ],
  plugins: [
    {
      name: '@electron-forge/plugin-auto-unpack-natives',
      config: {},
    },
  ],
};
```

## 4. GENERAR ELS EXECUTABLES (Build)

### Per a Linux (.deb i .zip)

```bash
npm run make
```

Resultat: Carpeta `out/make/deb/x64/`.

### Per a Windows Portable (.zip)

```bash
npm run make -- --platform=win32 --arch=x64
```

(Nota: Els guions `--` separats són necessaris). Resultat: Carpeta `out/make/zip/win32/x64/`.

## 🚨 CHECKLIST DE POSSIBLES ERRORS (Si alguna cosa falla)

### Pantalla Blanca?
- Has posat `base: './'` al `vite.config`?
- Has posat `createWebHashHistory` al router?
- Has tornat a fer `npm run build` després de canviar això?

### Error "Could not find binary" al fer el build?
- Revisa el `package.json`. El `name` i `productName` han de ser iguals, sense espais i en minúscules (ex: `projecte-examen`).
- Esborra la carpeta `out` (`rm -rf out`) i torna a provar.

### Error "rpmbuild not found"?
- Ves a `forge.config.js` i esborra tot el bloc `{ name: ...maker-rpm... }`.

### Error "You must install Mono/Wine"?
- Estàs intentant crear un instal·lador .exe (Squirrel). No ho facis.
- Fes només el ZIP per a Windows (Pas 4.2 d'aquesta guia).

### Error de connexió al Backend?
- Recorda que Electron és `file://`.
- Al `main.js` has de tenir `webSecurity: false`.
- Al frontend, la URL del backend ha de ser absoluta (`http://localhost:3000` o la IP), no relativa (`/api`).