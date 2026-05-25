# Control-de-regal-as-internas-
Dashboard de control mensual de regalías del personal: actividades, consumo interno, reconocimientos por Centro de Distribución.
import { useState, useMemo } from "react";

const CENTERS = ["Centro Norte", "Centro Sur", "Centro Este", "Centro Oeste", "Sede Central"];
const MONTHS = ["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
const TIPOS = ["Actividad", "Caja", "Reconocimiento"];
const TIPO_ICONS = { "Actividad": "🎯", "Caja": "📦", "Reconocimiento": "🏅" };
const TIPO_COLORS = {
  "Actividad": { bg: "#1a3a2a", accent: "#2ecc71", light: "#0f2318" },
  "Caja": { bg: "#1a2a3a", accent: "#3498db", light: "#0f1823" },
  "Reconocimiento": { bg: "#3a2a1a", accent: "#f39c12", light: "#231708" },
};

function randomBetween(a, b) { return Math.floor(Math.random() * (b - a + 1)) + a; }

function generateInitialData() {
  const data = [];
  let id = 1;
  CENTERS.forEach(centro => {
    MONTHS.forEach((mes, mesIdx) => {
      TIPOS.forEach(tipo => {
        const count = randomBetween(1, 8);
        for (let i = 0; i < count; i++) {
          data.push({
            id: id++,
            centro,
            mes: mesIdx,
            tipo,
            empleado: `Empleado ${randomBetween(1, 50)}`,
            descripcion: tipo === "Actividad"
              ? ["Taller creativo", "Evento deportivo", "Charla motivacional", "Team building"][randomBetween(0,3)]
              : tipo === "Caja"
              ? ["Caja navideña", "Cumpleaños", "Canasta básica", "Kit especial"][randomBetween(0,3)]
              : ["Empleado del mes", "Mejor desempeño", "Innovación", "Liderazgo"][randomBetween(0,3)],
            valor: randomBetween(500, 5000),
            fecha: `${randomBetween(1,28).toString().padStart(2,'0')}/${(mesIdx+1).toString().padStart(2,'0')}/2025`,
          });
        }
      });
    });
  });
  return data;
}

const INITIAL_DATA = generateInitialData();

const CURRENT_MONTH = 4; // Mayo index

export default function App() {
  const [data, setData] = useState(INITIAL_DATA);
  const [selectedMes, setSelectedMes] = useState(CURRENT_MONTH);
  const [selectedCentro, setSelectedCentro] = useState("Todos");
  const [selectedTipo, setSelectedTipo] = useState("Todos");
  const [showModal, setShowModal] = useState(false);
  const [activeTab, setActiveTab] = useState("overview");
  const [form, setForm] = useState({ centro: CENTERS[0], tipo: TIPOS[0], empleado: "", descripcion: "", valor: "", fecha: "" });

  const filtered = useMemo(() => data.filter(d =>
    (selectedCentro === "Todos" || d.centro === selectedCentro) &&
    (selectedTipo === "Todos" || d.tipo === selectedTipo) &&
    d.mes === selectedMes
  ), [data, selectedMes, selectedCentro, selectedTipo]);

  const allMes = useMemo(() => data.filter(d => d.mes === selectedMes), [data, selectedMes]);

  const statsByTipo = useMemo(() => TIPOS.map(tipo => ({
    tipo,
    count: allMes.filter(d => d.tipo === tipo).length,
    valor: allMes.filter(d => d.tipo === tipo).reduce((s, d) => s + d.valor, 0),
  })), [allMes]);

  const statsByCentro = useMemo(() => CENTERS.map(centro => ({
    centro,
    total: allMes.filter(d => d.centro === centro).length,
    valor: allMes.filter(d => d.centro === centro).reduce((s, d) => s + d.valor, 0),
    actividades: allMes.filter(d => d.centro === centro && d.tipo === "Actividad").length,
    cajas: allMes.filter(d => d.centro === centro && d.tipo === "Caja").length,
    reconocimientos: allMes.filter(d => d.centro === centro && d.tipo === "Reconocimiento").length,
  })), [allMes]);

  const monthlyTrend = useMemo(() => MONTHS.map((m, i) => ({
    mes: m,
    total: data.filter(d => d.mes === i).length,
    valor: data.filter(d => d.mes === i).reduce((s, d) => s + d.valor, 0),
  })), [data]);

  const maxTrend = Math.max(...monthlyTrend.map(m => m.total), 1);
  const maxValor = Math.max(...statsByCentro.map(c => c.valor), 1);
  const totalMes = allMes.length;
  const totalValorMes = allMes.reduce((s, d) => s + d.valor, 0);

  function handleAdd(e) {
    e.preventDefault();
    if (!form.empleado || !form.descripcion || !form.valor) return;
    setData(prev => [...prev, {
      id: prev.length + 1, ...form,
      mes: selectedMes, valor: parseInt(form.valor),
      fecha: form.fecha || `01/${(selectedMes+1).toString().padStart(2,'0')}/2025`,
    }]);
    setShowModal(false);
    setForm({ centro: CENTERS[0], tipo: TIPOS[0], empleado: "", descripcion: "", valor: "", fecha: "" });
  }

  function handleDelete(id) {
    setData(prev => prev.filter(d => d.id !== id));
  }

  return (
    <div style={{
      minHeight: "100vh",
      background: "#0a0f0a",
      fontFamily: "'DM Sans', 'Segoe UI', sans-serif",
      color: "#e8f5e9",
      padding: "0",
    }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600;700&family=Space+Mono:wght@400;700&display=swap');
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { width: 5px; height: 5px; }
        ::-webkit-scrollbar-track { background: #0a0f0a; }
        ::-webkit-scrollbar-thumb { background: #1e3a2a; border-radius: 10px; }
        .tab-btn { transition: all 0.2s; }
        .tab-btn:hover { background: #1a2e1e !important; }
        .row-item { transition: background 0.15s; }
        .row-item:hover { background: #141f15 !important; }
        .btn-primary { transition: all 0.2s; cursor: pointer; }
        .btn-primary:hover { filter: brightness(1.2); transform: translateY(-1px); }
        .month-btn { transition: all 0.15s; cursor: pointer; }
        .month-btn:hover { background: #1e3a2a !important; }
        .center-bar { transition: width 0.6s cubic-bezier(.4,0,.2,1); }
        .trend-bar { transition: height 0.6s cubic-bezier(.4,0,.2,1); }
        select, input { outline: none; }
        select:focus, input:focus { border-color: #2ecc71 !important; }
        .modal-bg { animation: fadeIn 0.2s; }
        .modal-card { animation: slideUp 0.25s cubic-bezier(.4,0,.2,1); }
        @keyframes fadeIn { from { opacity:0 } to { opacity:1 } }
        @keyframes slideUp { from { transform:translateY(30px);opacity:0 } to { transform:translateY(0);opacity:1 } }
        .delete-btn { opacity: 0; transition: opacity 0.15s; cursor: pointer; }
        .row-item:hover .delete-btn { opacity: 1; }
      `}</style>

      {/* Header */}
      <div style={{ background: "linear-gradient(180deg,#0d1f10 0%,#0a0f0a 100%)", borderBottom: "1px solid #1a3a1a", padding: "20px 28px 0" }}>
        <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 16 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
            <div style={{ width: 36, height: 36, borderRadius: 10, background: "linear-gradient(135deg,#2ecc71,#27ae60)", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 18 }}>🎁</div>
            <div>
              <div style={{ fontFamily: "'Space Mono', monospace", fontSize: 17, fontWeight: 700, letterSpacing: "-0.5px", color: "#e8f5e9" }}>REGALÍAS PERSONAL</div>
              <div style={{ fontSize: 11, color: "#4caf50", letterSpacing: 2, textTransform: "uppercase" }}>Dashboard de Control · 2025</div>
            </div>
          </div>
          <button className="btn-primary" onClick={() => setShowModal(true)} style={{
            background: "linear-gradient(135deg,#2ecc71,#27ae60)", color: "#0a1a0a", border: "none",
            borderRadius: 10, padding: "9px 18px", fontWeight: 700, fontSize: 13, display: "flex", alignItems: "center", gap: 6,
          }}>
            <span style={{ fontSize: 16 }}>+</span> Registrar Regalía
          </button>
        </div>

        {/* Month selector */}
        <div style={{ display: "flex", gap: 4, overflowX: "auto", paddingBottom: 0 }}>
          {MONTHS.map((m, i) => (
            <button key={m} className="month-btn" onClick={() => setSelectedMes(i)} style={{
              padding: "7px 14px", borderRadius: "8px 8px 0 0", border: "none", cursor: "pointer",
              background: selectedMes === i ? "#1e4a2a" : "transparent",
              color: selectedMes === i ? "#2ecc71" : "#6e9e7e",
              fontWeight: selectedMes === i ? 700 : 400,
              borderBottom: selectedMes === i ? "2px solid #2ecc71" : "2px solid transparent",
              fontSize: 13, whiteSpace: "nowrap",
              fontFamily: "'Space Mono', monospace",
            }}>{m}</button>
          ))}
        </div>
      </div>

      <div style={{ padding: "24px 28px" }}>
        {/* KPI Cards */}
        <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(200px,1fr))", gap: 14, marginBottom: 24 }}>
          <KpiCard icon="🎁" label="Total Regalías" value={totalMes} sub={`en ${MONTHS[selectedMes]}`} accent="#2ecc71" />
          <KpiCard icon="💰" label="Valor Total" value={`$${totalValorMes.toLocaleString()}`} sub="inversión del mes" accent="#f39c12" />
          {statsByTipo.map(s => (
            <KpiCard key={s.tipo} icon={TIPO_ICONS[s.tipo]} label={s.tipo + "s"} value={s.count} sub={`$${s.valor.toLocaleString()}`} accent={TIPO_COLORS[s.tipo].accent} />
          ))}
        </div>

        {/* Tabs */}
        <div style={{ display: "flex", gap: 4, marginBottom: 20 }}>
          {[["overview","📊 Resumen"],["detalle","📋 Detalle"],["tendencia","📈 Tendencia"]].map(([k,l]) => (
            <button key={k} className="tab-btn" onClick={() => setActiveTab(k)} style={{
              padding: "8px 18px", borderRadius: 8, border: "none", cursor: "pointer",
              background: activeTab === k ? "#1e4a2a" : "#0f1a10",
              color: activeTab === k ? "#2ecc71" : "#6e9e7e",
              fontWeight: activeTab === k ? 600 : 400, fontSize: 13,
            }}>{l}</button>
          ))}
        </div>

        {activeTab === "overview" && (
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 18 }}>
            {/* Por Centro */}
            <div style={{ background: "#0d1a0e", border: "1px solid #1a3a1a", borderRadius: 14, padding: 20 }}>
              <div style={{ fontSize: 12, letterSpacing: 2, color: "#4caf50", textTransform: "uppercase", marginBottom: 16 }}>Por Centro de Trabajo</div>
              {statsByCentro.map(c => (
                <div key={c.centro} style={{ marginBottom: 14 }}>
                  <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 5 }}>
                    <span style={{ fontSize: 13, color: "#c8e6c9" }}>{c.centro}</span>
                    <div style={{ display: "flex", gap: 10, fontSize: 12 }}>
                      <span style={{ color: "#2ecc71" }}>🎯{c.actividades}</span>
                      <span style={{ color: "#3498db" }}>📦{c.cajas}</span>
                      <span style={{ color: "#f39c12" }}>🏅{c.reconocimientos}</span>
                      <span style={{ color: "#aaa", fontFamily: "monospace" }}>${c.valor.toLocaleString()}</span>
                    </div>
                  </div>
                  <div style={{ height: 6, background: "#1a2a1a", borderRadius: 4, overflow: "hidden" }}>
                    <div className="center-bar" style={{ height: "100%", width: `${(c.valor/maxValor)*100}%`, background: "linear-gradient(90deg,#2ecc71,#27ae60)", borderRadius: 4 }} />
                  </div>
                </div>
              ))}
            </div>

            {/* Por Tipo */}
            <div style={{ background: "#0d1a0e", border: "1px solid #1a3a1a", borderRadius: 14, padding: 20 }}>
              <div style={{ fontSize: 12, letterSpacing: 2, color: "#4caf50", textTransform: "uppercase", marginBottom: 16 }}>Distribución por Tipo</div>
              <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
                {statsByTipo.map(s => {
                  const pct = totalMes ? Math.round((s.count/totalMes)*100) : 0;
                  const c = TIPO_COLORS[s.tipo];
                  return (
                    <div key={s.tipo} style={{ background: c.light, border: `1px solid ${c.bg}`, borderRadius: 12, padding: "14px 16px" }}>
                      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 8 }}>
                        <span style={{ fontSize: 15 }}>{TIPO_ICONS[s.tipo]} <span style={{ fontSize: 14, color: c.accent, fontWeight: 600 }}>{s.tipo}s</span></span>
                        <span style={{ fontFamily: "'Space Mono',monospace", fontSize: 22, fontWeight: 700, color: c.accent }}>{s.count}</span>
                      </div>
                      <div style={{ height: 4, background: "#1a2a1a", borderRadius: 4, overflow: "hidden", marginBottom: 6 }}>
                        <div className="center-bar" style={{ height: "100%", width: `${pct}%`, background: c.accent, borderRadius: 4 }} />
                      </div>
                      <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, color: "#6e9e7e" }}>
                        <span>{pct}% del total</span>
                        <span>${s.valor.toLocaleString()}</span>
                      </div>
                    </div>
                  );
                })}
              </div>
            </div>
          </div>
        )}

        {activeTab === "detalle" && (
          <div style={{ background: "#0d1a0e", border: "1px solid #1a3a1a", borderRadius: 14, overflow: "hidden" }}>
            {/* Filtros */}
            <div style={{ padding: "14px 20px", borderBottom: "1px solid #1a3a1a", display: "flex", gap: 10, flexWrap: "wrap", alignItems: "center" }}>
              <span style={{ fontSize: 12, color: "#4caf50", letterSpacing: 1 }}>FILTRAR:</span>
              <Select value={selectedCentro} onChange={e => setSelectedCentro(e.target.value)} options={["Todos", ...CENTERS]} />
              <Select value={selectedTipo} onChange={e => setSelectedTipo(e.target.value)} options={["Todos", ...TIPOS]} />
              <span style={{ marginLeft: "auto", fontSize: 12, color: "#6e9e7e" }}>{filtered.length} registros</span>
            </div>
            {/* Table Header */}
            <div style={{ display: "grid", gridTemplateColumns: "100px 120px 140px 1fr 90px 90px 36px", gap: 0, padding: "10px 20px", fontSize: 11, color: "#4caf50", letterSpacing: 1.5, textTransform: "uppercase", borderBottom: "1px solid #1a3a1a" }}>
              <span>Fecha</span><span>Centro</span><span>Empleado</span><span>Descripción</span><span>Tipo</span><span style={{textAlign:"right"}}>Valor</span><span/>
            </div>
            <div style={{ maxHeight: 420, overflowY: "auto" }}>
              {filtered.length === 0 && <div style={{ padding: 40, textAlign: "center", color: "#4a6e4a" }}>Sin registros para este período</div>}
              {filtered.map(d => {
                const c = TIPO_COLORS[d.tipo];
                return (
                  <div key={d.id} className="row-item" style={{ display: "grid", gridTemplateColumns: "100px 120px 140px 1fr 90px 90px 36px", gap: 0, padding: "10px 20px", borderBottom: "1px solid #111a11", alignItems: "center" }}>
                    <span style={{ fontSize: 12, color: "#6e9e7e", fontFamily: "monospace" }}>{d.fecha}</span>
                    <span style={{ fontSize: 12, color: "#c8e6c9" }}>{d.centro.replace("Centro ","")}</span>
                    <span style={{ fontSize: 13, color: "#e8f5e9" }}>{d.empleado}</span>
                    <span style={{ fontSize: 12, color: "#a5d6a7" }}>{d.descripcion}</span>
                    <span style={{ fontSize: 12, color: c.accent }}>{TIPO_ICONS[d.tipo]} {d.tipo}</span>
                    <span style={{ fontSize: 13, fontFamily: "monospace", color: "#f0f0f0", textAlign: "right" }}>${d.valor.toLocaleString()}</span>
                    <span className="delete-btn" onClick={() => handleDelete(d.id)} style={{ color: "#e74c3c", fontSize: 16, textAlign: "center" }}>×</span>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {activeTab === "tendencia" && (
          <div style={{ display: "grid", gridTemplateColumns: "1fr 320px", gap: 18 }}>
            {/* Monthly trend chart */}
            <div style={{ background: "#0d1a0e", border: "1px solid #1a3a1a", borderRadius: 14, padding: 24 }}>
              <div style={{ fontSize: 12, letterSpacing: 2, color: "#4caf50", textTransform: "uppercase", marginBottom: 20 }}>Tendencia Mensual 2025</div>
              <div style={{ display: "flex", alignItems: "flex-end", gap: 8, height: 180 }}>
                {monthlyTrend.map((m, i) => (
                  <div key={m.mes} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", gap: 4 }}>
                    <span style={{ fontSize: 10, color: "#4caf50", fontFamily: "monospace" }}>{m.total}</span>
                    <div
                      className="trend-bar"
                      onClick={() => setSelectedMes(i)}
                      style={{
                        width: "100%", borderRadius: "4px 4px 0 0", cursor: "pointer",
                        height: `${Math.max((m.total/maxTrend)*140, 4)}px`,
                        background: i === selectedMes
                          ? "linear-gradient(180deg,#2ecc71,#27ae60)"
                          : "linear-gradient(180deg,#1e4a2a,#163620)",
                        border: i === selectedMes ? "1px solid #2ecc71" : "1px solid #1a3a1a",
                        transition: "all 0.2s",
                      }}
                    />
                    <span style={{ fontSize: 10, color: i === selectedMes ? "#2ecc71" : "#4a6e4a" }}>{m.mes}</span>
                  </div>
                ))}
              </div>
            </div>
            {/* Monthly summary */}
            <div style={{ background: "#0d1a0e", border: "1px solid #1a3a1a", borderRadius: 14, padding: 20 }}>
              <div style={{ fontSize: 12, letterSpacing: 2, color: "#4caf50", textTransform: "uppercase", marginBottom: 16 }}>Resumen Anual</div>
              {monthlyTrend.map((m, i) => (
                <div key={m.mes} onClick={() => setSelectedMes(i)} style={{
                  display: "flex", justifyContent: "space-between", padding: "8px 10px",
                  borderRadius: 8, marginBottom: 4, cursor: "pointer",
                  background: i === selectedMes ? "#1e4a2a" : "transparent",
                  border: `1px solid ${i === selectedMes ? "#2ecc71" : "transparent"}`,
                }}>
                  <span style={{ fontSize: 13, color: i === selectedMes ? "#2ecc71" : "#a5d6a7" }}>{MONTHS[i]}</span>
                  <span style={{ fontSize: 13, color: "#6e9e7e" }}>{m.total} regalías</span>
                  <span style={{ fontSize: 12, fontFamily: "monospace", color: "#4caf50" }}>${m.valor.toLocaleString()}</span>
                </div>
              ))}
            </div>
          </div>
        )}
      </div>

      {/* Modal */}
      {showModal && (
        <div className="modal-bg" onClick={() => setShowModal(false)} style={{
          position: "fixed", inset: 0, background: "rgba(0,10,0,0.85)", display: "flex", alignItems: "center", justifyContent: "center", zIndex: 100,
        }}>
          <div className="modal-card" onClick={e => e.stopPropagation()} style={{
            background: "#0d1a0e", border: "1px solid #2ecc71", borderRadius: 18, padding: 32, width: 440, maxWidth: "95vw",
          }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 24 }}>
              <div style={{ fontFamily: "'Space Mono',monospace", fontSize: 16, color: "#2ecc71", fontWeight: 700 }}>🎁 Nueva Regalía</div>
              <button onClick={() => setShowModal(false)} style={{ background: "none", border: "none", color: "#4caf50", fontSize: 22, cursor: "pointer" }}>×</button>
            </div>
            <form onSubmit={handleAdd}>
              <FormRow label="Centro">
                <Select value={form.centro} onChange={e => setForm(f => ({...f, centro: e.target.value}))} options={CENTERS} full />
              </FormRow>
              <FormRow label="Tipo">
                <Select value={form.tipo} onChange={e => setForm(f => ({...f, tipo: e.target.value}))} options={TIPOS} full />
              </FormRow>
              <FormRow label="Empleado">
                <Input placeholder="Nombre del empleado" value={form.empleado} onChange={e => setForm(f => ({...f, empleado: e.target.value}))} required />
              </FormRow>
              <FormRow label="Descripción">
                <Input placeholder="Descripción de la regalía" value={form.descripcion} onChange={e => setForm(f => ({...f, descripcion: e.target.value}))} required />
              </FormRow>
              <FormRow label="Valor ($)">
                <Input type="number" placeholder="0" value={form.valor} onChange={e => setForm(f => ({...f, valor: e.target.value}))} required />
              </FormRow>
              <FormRow label="Fecha">
                <Input type="date" value={form.fecha} onChange={e => setForm(f => ({...f, fecha: e.target.value}))} />
              </FormRow>
              <button type="submit" className="btn-primary" style={{
                width: "100%", marginTop: 20, padding: "13px", background: "linear-gradient(135deg,#2ecc71,#27ae60)",
                border: "none", borderRadius: 12, color: "#0a1a0a", fontWeight: 700, fontSize: 15, cursor: "pointer",
              }}>Guardar Regalía</button>
            </form>
          </div>
        </div>
      )}
    </div>
  );
}

function KpiCard({ icon, label, value, sub, accent }) {
  return (
    <div style={{ background: "#0d1a0e", border: `1px solid #1a3a1a`, borderRadius: 14, padding: "18px 20px", position: "relative", overflow: "hidden" }}>
      <div style={{ position: "absolute", top: -10, right: -10, fontSize: 48, opacity: 0.06 }}>{icon}</div>
      <div style={{ fontSize: 20, marginBottom: 4 }}>{icon}</div>
      <div style={{ fontSize: 11, color: "#4caf50", letterSpacing: 1.5, textTransform: "uppercase", marginBottom: 4 }}>{label}</div>
      <div style={{ fontFamily: "'Space Mono',monospace", fontSize: 26, fontWeight: 700, color: accent }}>{value}</div>
      <div style={{ fontSize: 11, color: "#6e9e7e", marginTop: 2 }}>{sub}</div>
    </div>
  );
}

function Select({ value, onChange, options, full }) {
  return (
    <select value={value} onChange={onChange} style={{
      background: "#111f12", border: "1px solid #1e3a1e", borderRadius: 8,
      color: "#c8e6c9", padding: "9px 12px", fontSize: 13,
      width: full ? "100%" : "auto", cursor: "pointer",
    }}>
      {options.map(o => <option key={o} value={o}>{o}</option>)}
    </select>
  );
}

function Input({ ...props }) {
  return (
    <input {...props} style={{
      background: "#111f12", border: "1px solid #1e3a1e", borderRadius: 8,
      color: "#e8f5e9", padding: "9px 12px", fontSize: 13, width: "100%",
    }} />
  );
}

function FormRow({ label, children }) {
  return (
    <div style={{ marginBottom: 14 }}>
      <div style={{ fontSize: 11, color: "#4caf50", letterSpacing: 1, textTransform: "uppercase", marginBottom: 6 }}>{label}</div>
      {children}
    </div>
  );
}
