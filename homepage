import { useEffect, useMemo, useState } from "react";
import {
  LayoutDashboard,
  Radio,
  Map,
  Monitor,
  Activity,
  CalendarDays,
  Music,
  Server,
  FileText,
  Settings,
  Bell,
  Square,
  AlertTriangle,
  Volume2,
  Check,
} from "lucide-react";

/**
 * IP PA System — Dashboard
 * ต้องการ: react, lucide-react, tailwindcss (v3+)
 *   npm i lucide-react
 *
 * ฟอนต์ (IBM Plex Sans Thai + JetBrains Mono) จะถูกโหลดอัตโนมัติจาก Google Fonts
 * ถ้าต้องการให้ทำงานแบบออฟไลน์ ให้ดาวน์โหลดฟอนต์มาไว้ในโปรเจกต์แทน
 */

/* ───────────── ข้อมูลตัวอย่าง (แทนที่ด้วย API/WebSocket จริง) ───────────── */
const ZONES = [
  { id: 1, name: "โซน หมู่ที่ 1", devices: 1, online: 1 },
  { id: 2, name: "โซน หมู่ที่ 2", devices: 1, online: 1 },
];

const DEVICES = [
  {
    id: "582A8DD2F210",
    name: "PA1-ZONE_B",
    zone: "โซน หมู่ที่ 2",
    ip: "cloud",
    lastSeen: "0 วินาทีที่แล้ว",
    amp: true,
    online: true,
  },
  {
    id: "F0161D52BBFC",
    name: "PA1-ZONE_A",
    zone: "โซน หมู่ที่ 1",
    ip: "172.16.1.58",
    lastSeen: "1 วินาทีที่แล้ว",
    amp: true,
    online: true,
  },
];

const NAV = [
  { label: "แดชบอร์ด", icon: LayoutDashboard },
  { label: "กระจายเสียงสด", icon: Radio },
  { label: "จัดการโซน", icon: Map },
  { label: "อุปกรณ์", icon: Monitor },
  { label: "มอนิเตอร์เสียง", icon: Activity },
  { label: "ตารางเวลา", icon: CalendarDays },
  { label: "คลังเสียง", icon: Music },
  { label: "ผังเครือข่าย", icon: Server },
  { label: "บันทึกเหตุการณ์", icon: FileText },
  { label: "สถานะระบบ", icon: Activity },
  { label: "ตั้งค่า", icon: Settings },
];

/* ───────────── helpers ───────────── */
const pad = (n) => String(n).padStart(2, "0");
const fmtClock = (d) => `${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`;
const fmtDuration = (sec) =>
  `${pad(Math.floor(sec / 3600))}:${pad(Math.floor((sec % 3600) / 60))}:${pad(sec % 60)}`;
const dbToPct = (db) => Math.min(100, Math.max(0, ((db + 60) / 60) * 100));

/* ───────────── UI ย่อย ───────────── */
const Panel = ({ className = "", children }) => (
  <section className={`rounded-xl border border-[#1f2d44] bg-[#111b2d] ${className}`}>{children}</section>
);

const PanelTitle = ({ icon: Icon, children, right }) => (
  <div className="flex items-center justify-between">
    <h2 className="flex items-center gap-2 text-[11px] font-medium tracking-wide text-slate-400">
      {Icon && <Icon size={13} className="text-slate-500" />}
      {children}
    </h2>
    {right}
  </div>
);

const Badge = ({ tone = "green", children }) => {
  const tones = {
    green: "border-emerald-500/30 bg-emerald-500/10 text-emerald-400",
    red: "border-red-500/30 bg-red-500/10 text-red-400",
    blue: "border-blue-500/30 bg-blue-500/10 text-blue-400",
  };
  return (
    <span
      className={`inline-flex items-center gap-1.5 rounded-full border px-2 py-0.5 text-[10px] font-semibold ${tones[tone]}`}
    >
      <span className="h-1.5 w-1.5 rounded-full bg-current" />
      {children}
    </span>
  );
};

const StatCard = ({ label, value, sub, accent = "green", valueClass = "" }) => {
  const accents = {
    green: "border-l-emerald-500 text-emerald-400",
    red: "border-l-red-500 text-red-500",
    slate: "border-l-slate-500 text-slate-100",
  };
  return (
    <div
      className={`rounded-lg border border-[#1f2d44] border-l-[3px] bg-[#111b2d] px-3.5 py-3 ${accents[accent].split(" ")[0]}`}
    >
      <div className="text-[9px] text-slate-500">{label}</div>
      <div className={`mt-1 text-[22px] font-extrabold leading-tight ${valueClass || accents[accent].split(" ")[1]}`}>
        {value}
      </div>
      <div className="mt-0.5 text-[9px] text-slate-500">{sub}</div>
    </div>
  );
};

const Checkbox = ({ checked }) => (
  <span
    className={`flex h-[18px] w-[18px] items-center justify-center rounded border ${
      checked ? "border-blue-500 bg-blue-500" : "border-slate-600 bg-transparent"
    }`}
  >
    {checked && <Check size={12} strokeWidth={3} className="text-white" />}
  </span>
);

const StopButton = ({ onClick, className = "", children }) => (
  <button
    onClick={onClick}
    className={`inline-flex items-center justify-center gap-2 rounded-lg bg-red-500 font-bold text-white transition hover:bg-red-400 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-red-300 ${className}`}
  >
    <Square size={12} />
    {children}
  </button>
);

const VUBar = ({ channel, rms, peak }) => (
  <div className="flex items-center gap-3">
    <span className="w-3 text-[11px] font-semibold text-slate-300">{channel}</span>
    <div className="relative h-3.5 flex-1 overflow-hidden rounded-sm bg-[#0b1322]">
      <div
        className="h-full rounded-sm bg-gradient-to-r from-green-500 via-green-500 to-yellow-500 transition-[width] duration-100"
        style={{ width: `${dbToPct(rms)}%` }}
      />
      <div
        className="absolute top-0 h-full w-[2px] bg-white transition-[left] duration-100"
        style={{ left: `${dbToPct(peak)}%` }}
      />
    </div>
    <span className="w-10 text-right font-mono text-[10px] text-slate-400">{rms.toFixed(1)}</span>
  </div>
);

const VU_TICKS = [-60, -50, -40, -30, -20, -10, -6, -3, 0];

/* ───────────── หน้าหลัก ───────────── */
export default function PADashboard() {
  const [now, setNow] = useState(new Date());
  const [onAir, setOnAir] = useState(true);
  const [selected, setSelected] = useState(() => new Set(ZONES.map((z) => z.id)));
  const [elapsed, setElapsed] = useState(42 * 60 + 45); // วินาทีที่ออกอากาศมาแล้ว (ตัวอย่าง)
  const [levels, setLevels] = useState({ l: -15.2, r: -15.2, peakL: -1.5, peakR: -1.5 });

  /* โหลดฟอนต์ */
  useEffect(() => {
    const link = document.createElement("link");
    link.rel = "stylesheet";
    link.href =
      "https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+Thai:wght@400;500;600;700&family=JetBrains+Mono:wght@500;700&display=swap";
    document.head.appendChild(link);
    return () => link.remove();
  }, []);

  /* นาฬิกา + ระยะเวลาออกอากาศ */
  useEffect(() => {
    const t = setInterval(() => {
      setNow(new Date());
      if (onAir) setElapsed((s) => s + 1);
    }, 1000);
    return () => clearInterval(t);
  }, [onAir]);

  /* จำลองระดับเสียง ~7 ครั้ง/วินาที (แทนที่ด้วยข้อมูลจริงจาก pa-stream) */
  useEffect(() => {
    if (!onAir) {
      setLevels({ l: -60, r: -60, peakL: -60, peakR: -60 });
      return;
    }
    const t = setInterval(() => {
      setLevels((p) => {
        const l = -19 + Math.random() * 7;
        const r = -19 + Math.random() * 7;
        return {
          l,
          r,
          peakL: Math.max(l + 3, p.peakL - 0.8),
          peakR: Math.max(r + 3, p.peakR - 0.8),
        };
      });
    }, 140);
    return () => clearInterval(t);
  }, [onAir]);

  const allSelected = selected.size === ZONES.length;
  const selectedZones = useMemo(() => ZONES.filter((z) => selected.has(z.id)), [selected]);
  const totalDevices = selectedZones.reduce((a, z) => a + z.devices, 0);
  const totalOnline = selectedZones.reduce((a, z) => a + z.online, 0);

  const toggleZone = (id) =>
    setSelected((prev) => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  const toggleAll = () => setSelected(allSelected ? new Set() : new Set(ZONES.map((z) => z.id)));

  const startBroadcast = () => {
    if (selected.size === 0) return;
    setElapsed(0);
    setOnAir(true);
  };
  const stopBroadcast = () => setOnAir(false);

  const targetLabel = allSelected ? "ทุกโซน" : selectedZones.map((z) => z.name).join(", ") || "-";

  return (
    <div className="flex min-h-screen bg-[#0a1120] font-['IBM_Plex_Sans_Thai',sans-serif] text-slate-200">
      {/* ───── Sidebar ───── */}
      <aside className="sticky top-0 flex h-screen w-[197px] shrink-0 flex-col border-r border-[#1f2d44] bg-[#0d1626]">
        <div className="flex items-center gap-2.5 border-b border-[#1f2d44] px-3.5 py-3.5">
          <div className="flex h-9 w-9 items-center justify-center rounded-lg border border-blue-500/30 bg-blue-500/15">
            <Volume2 size={17} className="text-blue-400" />
          </div>
          <div className="leading-tight">
            <div className="text-[13px] font-bold text-slate-100">IP PA SYSTEM</div>
            <div className="text-[8px] text-slate-500">ระบบหอกระจายข่าว IP</div>
          </div>
        </div>

        <nav className="flex-1 space-y-0.5 px-2 py-3">
          {NAV.map(({ label, icon: Icon }, i) => {
            const active = i === 0;
            return (
              <a
                key={label}
                href="#"
                className={`relative flex items-center gap-2.5 rounded-lg px-3 py-2 text-[12px] transition ${
                  active
                    ? "border border-[#25365a] bg-[#16233f] font-semibold text-slate-100"
                    : "border border-transparent text-slate-400 hover:bg-[#121e35] hover:text-slate-200"
                }`}
              >
                {active && <span className="absolute -left-2 top-1.5 h-6 w-[3px] rounded-r bg-blue-500" />}
                <Icon size={15} className={active ? "text-blue-400" : "text-slate-500"} />
                {label}
              </a>
            );
          })}
        </nav>

        <div className="space-y-1.5 border-t border-[#1f2d44] px-3.5 py-3 text-[9px] text-slate-500">
          <div className="flex justify-between">
            <span>เซิร์ฟเวอร์</span>
            <span className="flex items-center gap-1 text-slate-300">
              <span className="h-1.5 w-1.5 rounded-full bg-emerald-500" /> Online
            </span>
          </div>
          <div className="flex justify-between">
            <span>อุปกรณ์ออนไลน์</span>
            <span className="font-semibold text-slate-300">2 / 2</span>
          </div>
          <div className="flex justify-between">
            <span>เวอร์ชัน</span>
            <span className="font-semibold text-slate-300">v2.0</span>
          </div>
        </div>
      </aside>

      {/* ───── Main ───── */}
      <div className="flex min-w-0 flex-1 flex-col">
        {/* Header */}
        <header className="flex h-[54px] items-center justify-between border-b border-[#1f2d44] bg-[#0d1626] px-5">
          <h1 className="text-[14px] font-semibold text-slate-100">แดชบอร์ด</h1>
          <div className="flex items-center gap-3">
            <div className="rounded-md border border-[#1f2d44] bg-black/60 px-3 py-1 font-['JetBrains_Mono',monospace] text-[20px] font-bold tracking-wider text-amber-400">
              {fmtClock(now)}
            </div>
            {onAir && (
              <span className="flex items-center gap-1.5 rounded-md border border-red-500/60 bg-red-500/10 px-2.5 py-1.5 text-[10px] font-bold text-red-400">
                <span className="h-1.5 w-1.5 animate-pulse rounded-full bg-red-500" />
                ON AIR
              </span>
            )}
            <StopButton onClick={stopBroadcast} className="px-3 py-1.5 text-[10px]">
              STOP ALL
            </StopButton>
            <button className="rounded-md border border-[#1f2d44] p-2 text-slate-400 hover:text-slate-200" aria-label="การแจ้งเตือน">
              <Bell size={14} />
            </button>
            <div className="flex items-center gap-2 text-[10px] text-slate-400">
              Operator
              <span className="flex h-6 w-6 items-center justify-center rounded-full bg-blue-500/20 text-[9px] font-bold text-blue-300">
                OP
              </span>
            </div>
          </div>
        </header>

        <main className="mx-auto w-full max-w-[1162px] space-y-3.5 px-4 py-4">
          {/* Stat cards */}
          <div className="grid grid-cols-2 gap-3 md:grid-cols-3 xl:grid-cols-6">
            <StatCard label="สถานะระบบ" value="NORMAL" sub="ระบบพร้อมใช้งาน" accent="green" />
            <StatCard
              label="การกระจายเสียง"
              value={onAir ? "ON AIR" : "STANDBY"}
              sub={onAir ? targetLabel : "ไม่มีการออกอากาศ"}
              accent={onAir ? "red" : "slate"}
            />
            <StatCard label="อุปกรณ์ออนไลน์" value="2 / 2" sub="ONLINE / TOTAL" accent="green" />
            <StatCard label="ออฟไลน์" value="0" sub="OFFLINE DEVICES" accent="green" />
            <StatCard label="โซนทั้งหมด" value={ZONES.length} sub="ZONES" accent="slate" />
            <StatCard label="เซิร์ฟเวอร์" value="ONLINE" sub="PA SERVER" accent="green" />
          </div>

          {/* On-air banner */}
          {onAir && (
            <div className="flex items-center justify-between rounded-xl border border-red-500/70 bg-gradient-to-r from-[#2a0f17] to-[#1f1019] px-5 py-3.5">
              <div className="flex items-center gap-10">
                <div className="flex items-center gap-3">
                  <span className="h-3 w-3 animate-pulse rounded-full bg-red-500 shadow-[0_0_10px_rgba(239,68,68,0.9)]" />
                  <div className="leading-tight">
                    <div className="text-[20px] font-extrabold tracking-wide text-white">ON AIR</div>
                    <div className="text-[8px] tracking-wider text-red-300/80">LIVE BROADCAST</div>
                  </div>
                </div>
                <div>
                  <div className="text-[8px] text-red-300/80">ปลายทาง</div>
                  <div className="text-[13px] font-bold text-white">{targetLabel}</div>
                </div>
                <div>
                  <div className="text-[8px] text-red-300/80">ระยะเวลา</div>
                  <div className="font-['JetBrains_Mono',monospace] text-[14px] font-bold text-white">
                    {fmtDuration(elapsed)}
                  </div>
                </div>
                <div>
                  <div className="text-[8px] text-red-300/80">อุปกรณ์ออกอากาศ</div>
                  <div className="text-[14px] font-bold text-white">
                    {totalOnline} / {totalDevices}
                  </div>
                </div>
              </div>
              <StopButton onClick={stopBroadcast} className="px-5 py-3 text-[12px]">
                STOP BROADCAST
              </StopButton>
            </div>
          )}

          {/* Control + VU */}
          <div className="grid grid-cols-1 gap-3.5 lg:grid-cols-[1.1fr_1fr]">
            {/* Live broadcast control */}
            <Panel className="p-4">
              <PanelTitle icon={Radio}>ควบคุมการกระจายเสียง · LIVE BROADCAST CONTROL</PanelTitle>

              <div className="mt-3.5 grid grid-cols-3 gap-2.5">
                {/* ALL ZONES */}
                <button
                  onClick={toggleAll}
                  className={`rounded-lg border p-3 text-left transition ${
                    allSelected
                      ? "border-blue-500 bg-[#13294d]"
                      : "border-[#25365a] bg-[#0f1a2e] hover:border-blue-500/50"
                  }`}
                >
                  <div className="flex items-start justify-between">
                    <span className="flex items-center gap-1.5 text-[12px] font-bold text-slate-100">
                      <Radio size={12} className="text-blue-400" />
                      ALL ZONES
                    </span>
                    <Checkbox checked={allSelected} />
                  </div>
                  <div className="mt-2 text-[9px] text-slate-400">
                    {ZONES.reduce((a, z) => a + z.devices, 0)} อุปกรณ์ ·{" "}
                    <b className="text-slate-200">{ZONES.reduce((a, z) => a + z.online, 0)} ออนไลน์</b>
                  </div>
                  <span className="mt-2 inline-flex items-center gap-1 rounded-full border border-emerald-500/30 bg-emerald-500/10 px-2 py-0.5 text-[9px] font-semibold text-emerald-400">
                    <span className="h-1.5 w-1.5 rounded-full bg-emerald-400" />
                    พร้อม
                  </span>
                </button>

                {/* รายโซน */}
                {ZONES.map((z) => {
                  const isSel = selected.has(z.id);
                  const live = onAir && isSel;
                  return (
                    <button
                      key={z.id}
                      onClick={() => toggleZone(z.id)}
                      className={`rounded-lg border p-3 text-left transition ${
                        live
                          ? "border-red-500/80 bg-[#2a1220]"
                          : isSel
                          ? "border-blue-500/60 bg-[#13294d]"
                          : "border-[#25365a] bg-[#0f1a2e] hover:border-blue-500/50"
                      }`}
                    >
                      <div className="flex items-start justify-between">
                        <span className="text-[12px] font-bold text-slate-100">{z.name}</span>
                        <Checkbox checked={isSel} />
                      </div>
                      <div className="mt-2 text-[9px] text-slate-400">
                        {z.devices} อุปกรณ์ · <b className="text-slate-200">{z.online} ออนไลน์</b>
                      </div>
                      <span
                        className={`mt-2 inline-flex items-center gap-1 rounded-full border px-2 py-0.5 text-[9px] font-semibold ${
                          live
                            ? "border-red-500/40 bg-red-500/10 text-red-400"
                            : "border-emerald-500/30 bg-emerald-500/10 text-emerald-400"
                        }`}
                      >
                        <span className="h-1.5 w-1.5 rounded-full bg-current" />
                        {live ? "ON AIR" : "พร้อม"}
                      </span>
                    </button>
                  );
                })}
              </div>

              <div className="mt-4 flex items-center justify-between border-t border-[#1f2d44] pt-4">
                <div className="flex items-end gap-6">
                  <div>
                    <div className="mb-1 text-[8px] text-slate-500">เลือกแล้ว</div>
                    <div className="flex gap-1.5">
                      {selectedZones.length === 0 && <span className="text-[10px] text-slate-500">ยังไม่ได้เลือกโซน</span>}
                      {selectedZones.map((z) => (
                        <span
                          key={z.id}
                          className="rounded border border-[#2a4170] bg-[#13294d]/60 px-2 py-1 text-[9px] text-slate-300"
                        >
                          {z.name}
                        </span>
                      ))}
                    </div>
                  </div>
                  <div>
                    <div className="text-[8px] text-slate-500">อุปกรณ์</div>
                    <div className="text-[14px] font-bold text-slate-100">{totalDevices}</div>
                  </div>
                  <div>
                    <div className="text-[8px] text-slate-500">ออนไลน์</div>
                    <div className="text-[14px] font-bold text-emerald-400">{totalOnline}</div>
                  </div>
                </div>

                {onAir ? (
                  <StopButton onClick={stopBroadcast} className="px-6 py-3.5 text-[12px]">
                    STOP BROADCAST
                  </StopButton>
                ) : (
                  <button
                    onClick={startBroadcast}
                    disabled={selected.size === 0}
                    className="inline-flex items-center gap-2 rounded-lg bg-blue-600 px-6 py-3.5 text-[12px] font-bold text-white transition hover:bg-blue-500 disabled:cursor-not-allowed disabled:opacity-40"
                  >
                    <Radio size={13} />
                    START BROADCAST
                  </button>
                )}
              </div>
            </Panel>

            {/* Master VU + zone status */}
            <Panel className="p-4">
              <PanelTitle
                icon={Activity}
                right={
                  <span
                    className={`rounded border px-1.5 py-0.5 text-[9px] font-bold ${
                      onAir
                        ? "border-emerald-500/40 bg-emerald-500/10 text-emerald-400"
                        : "border-slate-600 text-slate-500"
                    }`}
                  >
                    {onAir ? "LIVE" : "IDLE"}
                  </span>
                }
              >
                มาสเตอร์ VU
              </PanelTitle>

              <div className="mt-4 space-y-2">
                <VUBar channel="L" rms={levels.l} peak={levels.peakL} />
                <VUBar channel="R" rms={levels.r} peak={levels.peakR} />
              </div>

              {/* scale: ให้ตรงกับพื้นที่ของแถบ (หัก label ซ้าย 24px / ค่าขวา 52px) */}
              <div className="relative ml-6 mr-[52px] mt-1.5 h-3">
                {VU_TICKS.map((t) => (
                  <span
                    key={t}
                    className="absolute -translate-x-1/2 font-mono text-[8px] text-slate-500"
                    style={{ left: `${dbToPct(t)}%` }}
                  >
                    {t}
                  </span>
                ))}
              </div>

              <p className="mt-3 text-[8px] text-slate-500">
                ระดับไมค์จริงจาก pa-stream (อัปเดต ~7 ครั้ง/วินาที) · แถบ = RMS, ขีดขาว = peak (dBFS)
              </p>

              <div className="mt-4">
                <PanelTitle icon={Map}>สถานะรายโซน</PanelTitle>
                <ul className="mt-2 divide-y divide-[#1f2d44]">
                  {ZONES.map((z) => {
                    const live = onAir && selected.has(z.id);
                    return (
                      <li key={z.id} className="flex items-center justify-between py-2.5">
                        <span className="flex items-center gap-2 text-[11px] font-semibold text-slate-100">
                          <span className={`h-1.5 w-1.5 rounded-full ${live ? "bg-red-500" : "bg-emerald-500"}`} />
                          {z.name}
                        </span>
                        <span className="flex items-center gap-3">
                          <span className="text-[10px] text-slate-500">
                            {z.online}/{z.devices} ออนไลน์
                          </span>
                          <Badge tone={live ? "red" : "green"}>{live ? "ON AIR" : "พร้อม"}</Badge>
                        </span>
                      </li>
                    );
                  })}
                </ul>
              </div>
            </Panel>
          </div>

          {/* Active alerts */}
          <Panel className="px-4 py-3.5">
            <PanelTitle icon={AlertTriangle} right={<Badge tone="green">ปกติ</Badge>}>
              การแจ้งเตือนที่กำลังเกิด · ACTIVE ALERTS
            </PanelTitle>
            <p className="py-5 text-center text-[10px] text-emerald-400/80">
              ✓ ไม่มีการแจ้งเตือน — ระบบทำงานปกติ
            </p>
          </Panel>

          {/* Recent devices */}
          <Panel className="px-4 py-3.5">
            <PanelTitle
              icon={Monitor}
              right={
                <button className="rounded-md border border-[#25365a] px-3 py-1.5 text-[10px] font-semibold text-slate-300 hover:bg-[#16233f]">
                  ดูทั้งหมด →
                </button>
              }
            >
              อุปกรณ์ล่าสุด
            </PanelTitle>

            <div className="mt-3 overflow-x-auto">
              <table className="w-full min-w-[640px] text-left text-[10px]">
                <thead>
                  <tr className="border-b border-[#1f2d44] text-[9px] font-medium text-slate-500">
                    <th className="px-2 py-2.5 font-medium">สถานะ</th>
                    <th className="px-2 py-2.5 font-medium">ชื่อ</th>
                    <th className="px-2 py-2.5 font-medium">โซน</th>
                    <th className="px-2 py-2.5 font-medium">IP</th>
                    <th className="px-2 py-2.5 font-medium">LAST SEEN</th>
                    <th className="px-2 py-2.5 font-medium">แอมป์</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-[#1f2d44]">
                  {DEVICES.map((d) => (
                    <tr key={d.id} className="hover:bg-[#13203a]">
                      <td className="px-2 py-3">
                        <span
                          className={`inline-flex items-center gap-1.5 rounded-full border px-2 py-0.5 text-[9px] font-semibold ${
                            d.online
                              ? "border-emerald-500/30 bg-emerald-500/10 text-emerald-400"
                              : "border-slate-600 text-slate-500"
                          }`}
                        >
                          <span className="h-1.5 w-1.5 rounded-full bg-current" />
                          {d.online ? "ONLINE" : "OFFLINE"}
                        </span>
                      </td>
                      <td className="px-2 py-3">
                        <div className="font-semibold text-slate-100">{d.name}</div>
                        <div className="font-mono text-[8px] text-slate-500">{d.id}</div>
                      </td>
                      <td className="px-2 py-3 text-slate-200">{d.zone}</td>
                      <td className="px-2 py-3 font-mono text-slate-300">{d.ip}</td>
                      <td className="px-2 py-3 text-slate-300">{d.lastSeen}</td>
                      <td className="px-2 py-3">
                        <span className="inline-flex items-center gap-1.5 font-semibold text-slate-200">
                          <span className={`h-1.5 w-1.5 rounded-full ${d.amp ? "bg-red-500" : "bg-slate-600"}`} />
                          {d.amp ? "เปิด" : "ปิด"}
                        </span>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </Panel>
        </main>
      </div>
    </div>
  );
}
