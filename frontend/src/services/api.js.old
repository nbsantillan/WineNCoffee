import axios from "axios";
//para rutas que usan api
//const api = axios.create({
//  baseURL: "http://localhost:5000/api" //ruta vieja no se si se usa pero la dejamos
//});

//para rutas directas al backend por ejemplo metrics
const api = axios.create({
  baseURL: "http://localhost:3000" //backend por defecto
});

// Adjuntar token si existe
api.interceptors.request.use((config) => {
  const t = localStorage.getItem("token");
  if (t) config.headers.Authorization = `Bearer ${t}`;
  return config;
});


export default api;
