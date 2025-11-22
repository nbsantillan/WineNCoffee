import React, { useEffect, useState, useRef } from "react";
import { useNavigate } from "react-router-dom";
import { searchProducts } from "../services/productService";
import "./searchBar.css";

function SearchBar() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [open, setOpen] = useState(false);
  const [loading, setLoading] = useState(false);

  const ref = useRef(null);
  const navigate = useNavigate();
  const debounceRef = useRef(null);

  // Llamada a backend con debounce
  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      setOpen(false);
      return;
    }

    // Evita spam de requests
    clearTimeout(debounceRef.current);
    debounceRef.current = setTimeout(async () => {
      try {
        setLoading(true);
        const data = await searchProducts(query);
        setResults(data);
        setOpen(true);
      } catch (err) {
        console.error("Error buscando productos:", err);
        setResults([]);
      } finally {
        setLoading(false);
      }
    }, 300);
  }, [query]);

  // Cerrar el dropdown al hacer click afuera
  useEffect(() => {
    const handleClick = (e) => {
      if (ref.current && !ref.current.contains(e.target)) {
        setOpen(false);
      }
    };

    document.addEventListener("mousedown", handleClick);
    return () => document.removeEventListener("mousedown", handleClick);
  }, []);

  const handleSelect = (id) => {
    navigate(`/product/${id}`);
    setQuery("");
    setOpen(false);
  };

  return (
    <div className="search-container" ref={ref}>
      <input
        type="text"
        placeholder="Buscar productos..."
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        onFocus={() => query.length > 0 && setOpen(true)}
        className="search-input"
      />

      {open && (
        <ul className="search-results">
          {/* Loader */}
          {loading && (
            <li className="loading">Buscando...</li>
          )}

          {/* Resultados */}
          {!loading &&
            results.length > 0 &&
            results.map((item) => (
              <li
                key={item.product_id}
                onClick={() => handleSelect(item.product_id)}
              >
                <img
                  src={
                    item.thumbnail?.startsWith("http")
                      ? item.thumbnail
                      : `http://localhost:3000/${item.thumbnail}`
                  }
                  alt={item.name}
                />
                <span>{item.name}</span>
              </li>
            ))}

          {/* Sin resultados */}
          {!loading && results.length === 0 && (
            <li className="no-results">No hay coincidencias</li>
          )}
        </ul>
      )}
    </div>
  );
}

export default SearchBar;
