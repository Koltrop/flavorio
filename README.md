# Vue 3D Model Viewer

Una aplicación Vue.js que carga y muestra un modelo 3D en la página principal usando Three.js.

## Características

- 🎨 Interfaz moderna y responsive
- 🎮 Controles interactivos para el modelo 3D
- 🔄 Rotación automática del modelo
- 📐 Vista wireframe opcional
- 🎯 Reset de cámara
- 💡 Iluminación y sombras realistas
- 📱 Diseño adaptable a diferentes tamaños de pantalla

## Instalación

1. Instala las dependencias:
```bash
npm install
```

2. Ejecuta el servidor de desarrollo:
```bash
npm run dev
```

3. Abre tu navegador en `http://localhost:3000`

## Scripts Disponibles

- `npm run dev` - Inicia el servidor de desarrollo
- `npm run build` - Construye la aplicación para producción
- `npm run preview` - Previsualiza la build de producción

## Tecnologías Utilizadas

- **Vue 3** - Framework de JavaScript progresivo
- **Vite** - Herramienta de construcción rápida
- **Three.js** - Biblioteca para gráficos 3D en WebGL

## Estructura del Proyecto

```
src/
├── App.vue          # Componente principal con el visor 3D
├── main.js          # Punto de entrada de la aplicación
└── style.css        # Estilos globales
```

## Controles

- **Resetear Cámara**: Vuelve la cámara a su posición inicial
- **Rotar/Pausar**: Controla la rotación automática del modelo
- **Sólido/Wireframe**: Cambia entre vista sólida y wireframe

## Personalización

Para cargar tu propio modelo 3D, puedes:

1. Reemplazar la geometría del cubo en `App.vue` con tu modelo
2. Usar `THREE.GLTFLoader` para cargar archivos GLTF/GLB
3. Ajustar la iluminación y materiales según tus necesidades

## Licencia

MIT 