import React, { useEffect, useState } from "react";
import { useCart } from "../context/CartContext";
import { useNavigate, useLocation } from "react-router-dom";
import { confirmCheckout } from "../services/checkoutService";
import "./Resume.css";

const Resume = () => {
  const { items, shippingCost, clear } = useCart();
  const [paymentConfirmed, setPaymentConfirmed] = useState(false);
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();
  const location = useLocation();

  // Detectar automáticamente si todos los pasos del checkout están completos
  useEffect(() => {
    const payment = localStorage.getItem("paymentMethod");
    const userData = JSON.parse(localStorage.getItem("userData")) || {};
    const envioOk = localStorage.getItem("shippingCost");

    if (payment && userData?.personalData && envioOk) {
      setPaymentConfirmed(true);
    } else {
      setPaymentConfirmed(false);
    }
  }, [location.pathname]); // se actualiza si cambia de paso

  // Recuperar datos del localStorage unificados desde Profile.jsx y Payment.jsx
  const getCheckoutData = () => {
    const storedUserData = JSON.parse(localStorage.getItem("userData")) || {};
    const { personalData = {}, shippingData = {}, envioLocal } = storedUserData;

    const metodoPago = localStorage.getItem("paymentMethod");
    const datosTarjetaCredito =
      JSON.parse(localStorage.getItem("datosTarjeta_credito")) || {};
    const datosTarjetaDebito =
      JSON.parse(localStorage.getItem("datosTarjeta_debito")) || {};

    let datosMetodo = null;
    if (metodoPago === "credito") datosMetodo = datosTarjetaCredito;
    else if (metodoPago === "debito") datosMetodo = datosTarjetaDebito;
    else if (metodoPago === "mercadopago") datosMetodo = { wallet: "mercadopago" };

    return {
      cliente: personalData,
      direccion: shippingData,
      metodoPago,
      datosMetodo,
      envioLocal,
    };
  };

  const subtotal = items.reduce(
    (acc, it) => acc + (Number(it.price) || 0) * (it.qty || 1),
    0
  );
  const finalTotal = shippingCost ? subtotal + shippingCost : subtotal;

  const handleConfirmPurchase = async () => {
    const { cliente, direccion, metodoPago, datosMetodo } = getCheckoutData();

    // 🧩 Validaciones previas
    if (
      !cliente?.dni ||
      !cliente?.nombre ||
      !cliente?.apellido ||
      !cliente?.email
    ) {
      alert(" Faltan datos del cliente.");
      return;
    }
    if (!direccion?.calle || !direccion?.ciudad || !direccion?.provincia) {
      alert(" Faltan datos de la dirección.");
      return;
    }
    if (!metodoPago) {
      alert(" Faltan los datos del método de pago.");
      return;
    }
    if (items.length === 0) {
      alert(" No hay productos en el carrito.");
      return;
    }

    setLoading(true);

    try {
      const direccionAdaptada = {
        street: direccion.calle + " " + (direccion.altura || ""),
        city: direccion.ciudad,
        province: direccion.provincia,
        country: "Argentina",
        postal_code: direccion.cp,
      };

      const clienteAdaptado = {
        first_name: cliente.nombre,
        last_name: cliente.apellido,
        email: cliente.email,
        dni: cliente.dni,
        phone: cliente.telefono,
      };

      const data = await confirmCheckout({
        cliente: clienteAdaptado,
        direccion: direccionAdaptada,
        carrito: items,
        metodoPago: { type: metodoPago, datos: datosMetodo },
        shippingCost,
        total: finalTotal,
      });

      if (data.success) {
        console.log("¡Compra confirmada con éxito!");
        clear();
        localStorage.removeItem("userData");
        localStorage.removeItem("paymentMethod");
        localStorage.removeItem("shippingCost");

        if (data.order_id) {
          navigate(`/order/${data.order_id}`, {
            state: { shippingCost }, 
          });
          
        } else {
          navigate("/", {replace: true });
        }
      } else {
        alert("Error al confirmar la compra: " + (data.message || "Desconocido"));
      }
    } catch (error) {
      console.error("Error al confirmar el checkout:", error);
      alert("Ocurrió un error al procesar tu compra.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="resumen-compra">
      <h3>Resumen de tu compra</h3>

      {items.length === 0 ? (
        <p>No hay productos en el carrito.</p>
      ) : (
        <ul className="resumen-lista">
          {items.map((item) => (
            <li key={item.id}>
              <div className="item-left">

          {/* MINIATURA */}
          <img
            className="resume-thumb"
            src={
              item.thumbnail?.startsWith("http")
                ? item.thumbnail
                : `http://localhost:3000/${item.thumbnail}`
            }
            alt={item.name || item.nombre}
          />

    <div className="item-info">
      <div className="item-title">{item.name}</div>
      <div className="item-qty">x{item.qty}</div>
    </div>
  </div>

  <div className="item-right">
    ${(Number(item.price) * (item.qty || 1)).toLocaleString()}
  </div>
</li>

          ))}
        </ul>
      )}

      <div className="subtotal-row">
        <span>Subtotal:</span>
        <strong>${subtotal.toLocaleString()}</strong>
      </div>

      <div className="envio-row">
        <span>Envío:</span>
        <strong>
          {shippingCost ? `$${shippingCost.toLocaleString()}` : "A calcular"}
        </strong>
      </div>

      <div className="total-row">
        <span>Total:</span>
        <strong>${finalTotal.toLocaleString()}</strong>
      </div>

      <div className="resumen-botones">
        <button className="btn-volver" onClick={() => navigate("/carrito")}>
          ← Volver al carrito
        </button>

        {location.pathname === "/checkout/confirmation" && paymentConfirmed && (
          <button
            className="btn-confirmar"
            onClick={handleConfirmPurchase}
            disabled={loading}
          >
            {loading ? "Procesando..." : "Confirmar compra"}
          </button>
        )}
      </div>
    </div>
  );
};

export default Resume;
