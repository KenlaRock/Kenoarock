import React, { useEffect, useMemo, useRef, useState } from "react";
import { motion, useMotionValue, useSpring, useTransform } from "framer-motion";
import {
  Search,
  Flame,
  Sun,
  Sparkles,
  Star,
  Clock,
  Filter,
  SortAsc,
  SortDesc,
  ChevronDown,
  History,
  Info,
} from "lucide-react";

/**
 * Icarus ⇄ Vega Chronology — Cinematic Timeline Site
 * Framework: React + TailwindCSS + Framer Motion
 * Style: black canvas, rounded-2xl, wide-kerning caps, monospaced labels
 * Theme: toggles between Icarus (burnt orange) and Vega (cyan star)
 * Notes: Minimal DOM, GPU-friendly effects, responsive from xs → 2xl
 */

type Character = "icarus" | "vega";

// THEME TOKENS (used via CSS variables so Tailwind can stay static)
const THEMES: Record<
  Character,
  {
    name: string;
    from: string;
    to: string;
    ring: string; // glow ring rgba
    border: string;
    tile: string;
    accent: string; // text accent hex
    accentSlider: string; // Tailwind class for input[type=range] accent
    icon: React.ReactNode;
  }
> = {
  icarus: {
    name: "Icarus",
    from: "#ea580c",
    to: "#f59e0b",
    ring: "rgba(234,88,12,0.35)",
    border: "rgba(124,45,18,0.6)",
    tile: "rgba(124,45,18,0.18)",
    accent: "#fb923c",
    accentSlider: "accent-orange-500",
    icon: <Flame className="h-4 w-4" aria-hidden />,
  },
  vega: {
    name: "Vega",
    from: "#06b6d4",
    to: "#22d3ee",
    ring: "rgba(34,211,238,0.35)",
    border: "rgba(8,145,178,0.6)",
    tile: "rgba(8,145,178,0.18)",
    accent: "#67e8f9",
    accentSlider: "accent-cyan-500",
    icon: <Star className="h-4 w-4" aria-hidden />,
  },
};

type Arc =
  | "Isolation"
  | "Confrontation"
  | "Awakening"
  | "Duality"
  | "Climax"
  | "Drift"
  | "Discovery"
  | "Convergence"
  | "Singularity";

type Scene = {
  id: string;
  title: string;
  arc: Arc;
  revealPct: number; // 0-100 — where in the narrative this is revealed
  statusPct?: number; // optional completion meter
  summary: string;
  dependencies?: string[];
  tags?: string[];
  date?: string;
};

const ICARUS_SCENES: Scene[] = [
  {
    id: "S00x",
    title: "Wax & Brine (Prologue)",
    arc: "Isolation",
    revealPct: 10,
    statusPct: 95,
    summary:
      "A shoreline of failed blueprints; feathers catalogued, wings unnamed. The sea keeps his first secret.",
    tags: ["workshop", "shore"],
  },
  {
    id: "S01",
    title: "A Scar Without Origin",
    arc: "Isolation",
    revealPct: 20,
    statusPct: 80,
    summary:
      "Vega wakes with an untraceable scar; the world is quiet enough to hear dust settle.",
    tags: ["vega:scar", "memory"],
  },
  {
    id: "S23f",
    title: "First Ascent (Flashback)",
    arc: "Awakening",
    revealPct: 40,
    statusPct: 85,
    summary:
      "Sunlight threads the wax; city roofs fall away. He learns the sky has a temperature.",
    tags: ["flashback", "flight:alpha"],
    date: "–",
  },
  {
    id: "S50",
    title: "Dödens Monolog",
    arc: "Confrontation",
    revealPct: 60,
    statusPct: 90,
    summary:
      "Icarus speaks to the dark, unspooling a ritual that erases the symbols he swore to protect.",
    tags: ["icarus:brand", "void:glyph"],
    dependencies: ["S01"],
  },
  {
    id: "S71x",
    title: "The Fall (Flashback)",
    arc: "Duality",
    revealPct: 75,
    statusPct: 70,
    summary:
      "Altitude is a covenant. Wax liquefies into signatures across the sea's black ledger.",
    tags: ["death:myth", "oath"],
  },
  {
    id: "S90",
    title: "The Cyan Star — Ultimatum",
    arc: "Duality",
    revealPct: 90,
    statusPct: 75,
    summary:
      "The link between them—The Cipher—shivers against fate; choice narrows to a blade.",
    tags: ["cipher:hint", "duality"],
    dependencies: ["S01", "S50"],
  },
  {
    id: "0717",
    title: "Scene 0717 (Final Reveal)",
    arc: "Climax",
    revealPct: 100,
    statusPct: 70,
    summary:
      "The sibling bond surfaces as Icarus burns; Vega is remade in the ash-storm of memory.",
    tags: ["cipher:reveal", "sacrifice"],
    dependencies: ["S01", "S50", "S90"],
  },
];

const VEGA_SCENES: Scene[] = [
  {
    id: "V-α",
    title: "Cryo-Observatory (Waking)",
    arc: "Discovery",
    revealPct: 15,
    statusPct: 90,
    summary:
      "A cyan aperture opens behind her eyelids; telemetry bleeds into language.",
    tags: ["vega:aperture", "telemetry"],
  },
  {
    id: "V-δ",
    title: "The Star That Watches",
    arc: "Drift",
    revealPct: 35,
    statusPct: 80,
    summary:
      "A solar twin hums in her bones; streetlights flicker in sympathy.",
    tags: ["cyan:star", "resonance"],
  },
  {
    id: "V-π",
    title: "Edge of Convergence",
    arc: "Convergence",
    revealPct: 60,
    statusPct: 65,
    summary:
      "She learns the Cipher can fold distance the way grief folds time.",
    tags: ["cipher", "fold"],
  },
  {
    id: "V-Ω",
    title: "Solar State (True Vega)",
    arc: "Singularity",
    revealPct: 95,
    statusPct: 55,
    summary:
      "The cyan star crowns the horizon of her chest; photons choose her first.",
    tags: ["solar:state", "ascension"],
  },
];

export default function IcarusVegaChronologySite() {
  const [character, setCharacter] = useState<Character>("icarus");
  const [q, setQ] = useState("");
  const [arc, setArc] = useState<Arc | "All">("All");
  const [minReveal, setMinReveal] = useState(0);
  const [sortAsc, setSortAsc] = useState(true);
  const theme = THEMES[character];

  // Pointer parallax for the hero orb
  const px = useMotionValue(0);
  const py = useMotionValue(0);
  const sx = useSpring(px, { stiffness: 60, damping: 15 });
  const sy = useSpring(py, { stiffness: 60, damping: 15 });
  const orbX = useTransform(sx, (v) => v * 0.03);
  const orbY = useTransform(sy, (v) => v * 0.03);

  const scenes = character === "icarus" ? ICARUS_SCENES : VEGA_SCENES;
  const items = useMemo(() => {
    let rows = [...scenes];
    if (arc !== "All") rows = rows.filter((r) => r.arc === arc);
    const search = q.trim().toLowerCase();
    if (search) {
      rows = rows.filter((r) =>
        [r.id, r.title, r.summary, r.arc, ...(r.tags || [])]
          .join(" ")
          .toLowerCase()
          .includes(search)
      );
    }
    rows = rows.filter((r) => r.revealPct >= minReveal);
    rows.sort((a, b) => (sortAsc ? a.revealPct - b.revealPct : b.revealPct - a.revealPct));
    return rows;
  }, [scenes, q, arc, minReveal, sortAsc]);

  // CSS variables for theme palette
  const cssVars: React.CSSProperties = {
    // @ts-ignore
    "--theme-from": theme.from,
    "--theme-to": theme.to,
    "--theme-ring": theme.ring,
    "--theme-border": theme.border,
    "--theme-tile": theme.tile,
    "--theme-accent": theme.accent,
  } as any;

  return (
    <div
      className="min-h-screen w-full bg-black text-white"
      style={cssVars}
      onPointerMove={(e) => {
        const { innerWidth, innerHeight } = window;
        px.set(e.clientX - innerWidth / 2);
        py.set(e.clientY - innerHeight / 2);
      }}
    >
      {/* BACKDROP: theme radial + grid + dust */}
      <Backdrop />

      {/* HERO */}
      <section className="relative px-6 md:px-10 lg:px-16 py-12 md:py-16">
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.6 }}
          className="mx-auto max-w-6xl"
        >
          <div className="flex flex-col md:flex-row md:items-end md:justify-between gap-8">
            <div>
              <div className="flex items-center gap-3" style={{ color: "var(--theme-accent)" }}>
                {theme.icon}
                <span className="font-mono text-xs tracking-widest uppercase">
                  {THEMES[character].name} // Timeline Module
                </span>
              </div>
              <h1 className="mt-3 text-3xl md:text-4xl lg:text-5xl font-extrabold tracking-[0.2em] uppercase">
                Chronology & Reveal Vector
              </h1>
              <p className="mt-4 max-w-3xl text-sm md:text-base opacity-80 font-mono">
                Toggle characters to shift palette and symbolism. Filter by arc, threshold by reveal, and read the ash—or starlight—for order.
              </p>

              <div className="mt-6">
                <CharacterToggle value={character} onChange={setCharacter} />
              </div>
            </div>

            {/* ORB VISUAL */}
            <motion.div style={{ x: orbX, y: orbY }} className="self-center md:self-end">
              <SolarOrb character={character} />
            </motion.div>
          </div>

          {/* Controls */}
          <div className="mt-8 grid grid-cols-1 md:grid-cols-3 gap-3">
            {/* Search */}
            <label
              className="relative flex items-center gap-2 rounded-2xl border bg-black/40 px-3 py-2"
              style={{ borderColor: "var(--theme-border)" }}
            >
              <Search className="h-4 w-4 opacity-80" aria-hidden />
              <input
                value={q}
                onChange={(e) => setQ(e.target.value)}
                placeholder="Search scenes, symbols, or IDs"
                className="w-full bg-transparent outline-none placeholder:opacity-60 font-mono text-sm"
                aria-label="Search timeline"
              />
            </label>

            {/* Arc Filter */}
            <div
              className="relative flex items-center rounded-2xl border bg-black/40 px-3 py-2"
              style={{ borderColor: "var(--theme-border)" }}
            >
              <Filter className="mr-2 h-4 w-4 opacity-80" aria-hidden />
              <select
                value={arc}
                onChange={(e) => setArc(e.target.value as any)}
                className="w-full bg-transparent outline-none font-mono text-sm uppercase tracking-widest appearance-none pr-6"
                aria-label="Filter by arc"
              >
                <option className="bg-black" value="All">All Arcs</option>
                {[
                  "Isolation",
                  "Confrontation",
                  "Awakening",
                  "Duality",
                  "Climax",
                  "Drift",
                  "Discovery",
                  "Convergence",
                  "Singularity",
                ].map((a) => (
                  <option className="bg-black" key={a} value={a}>
                    {a}
                  </option>
                ))}
              </select>
              <ChevronDown className="absolute right-3 h-4 w-4 opacity-60" aria-hidden />
            </div>

            {/* Sort & Reveal Threshold */}
            <div
              className="flex flex-col gap-2 rounded-2xl border bg-black/40 p-3"
              style={{ borderColor: "var(--theme-border)" }}
            >
              <div className="flex items-center justify-between">
                <div className="flex items-center gap-2">
                  <Clock className="h-4 w-4 opacity-80" aria-hidden />
                  <span className="font-mono text-xs uppercase tracking-widest">
                    Reveal ≥ {minReveal}%
                  </span>
                </div>
                <button
                  onClick={() => setSortAsc((v) => !v)}
                  className="inline-flex items-center gap-1 rounded-xl border px-2 py-1 text-xs font-mono uppercase tracking-widest hover:bg-white/5"
                  style={{ borderColor: "var(--theme-border)" }}
                  aria-label="Toggle sort order"
                >
                  {sortAsc ? (
                    <>
                      <SortAsc className="h-4 w-4" /> Asc
                    </>
                  ) : (
                    <>
                      <SortDesc className="h-4 w-4" /> Desc
                    </>
                  )}
                </button>
              </div>
              <input
                type="range"
                min={0}
                max={100}
                step={10}
                value={minReveal}
                onChange={(e) => setMinReveal(parseInt(e.target.value, 10))}
                className={THEMES[character].accentSlider}
                aria-label="Minimum reveal percentage"
              />
            </div>
          </div>
        </motion.div>
      </section>

      {/* LORE STRIP: background + flashbacks */}
      <section className="relative px-6 md:px-10 lg:px-16">
        <div className="mx-auto max-w-6xl grid md:grid-cols-5 gap-4">
          <LoreCard
            icon={<Info className="h-4 w-4" />}
            title="Background"
            body={
              character === "icarus"
                ? "Blueprints, salt, and the arithmetic of feathers; a boy translates heat into hope."
                : "Telemetry becomes instinct; Vega learns to read the galaxy the way others read a room."
            }
          />
          <LoreCard
            icon={<History className="h-4 w-4" />}
            title="Flashbacks"
            body={
              character === "icarus"
                ? "First ascent, the contract with sunlight, and the memory of an ocean that keeps good books."
                : "Stations asleep under auroras; a cyan hum that won’t fit inside any instrument."
            }
          />
          <div className="md:col-span-3">
            <EasterEdges>
              <div className="rounded-2xl border p-4 md:p-6 bg-black/40" style={{ borderColor: "var(--theme-border)" }}>
                <div className="flex items-center gap-2" style={{ color: "var(--theme-accent)" }}>
                  <Sparkles className="h-4 w-4" />
                  <span className="font-mono text-xs tracking-widest uppercase">Symbolic Hints</span>
                </div>
                <p className="mt-2 text-sm opacity-80">
                  Edges whisper: {character === "vega" ? "stellar coronas, cyan diffraction, cold fire" : "cinder motes, burnt sigils, solar residue"} — present but peripheral.
                </p>
              </div>
            </EasterEdges>
          </div>
        </div>
      </section>

      {/* TIMELINE */}
      <section className="relative px-6 md:px-10 lg:px-16 pb-24 pt-10">
        <div className="mx-auto max-w-6xl">
          <div className="relative">
            {/* Spine */}
            <div
              aria-hidden
              className="absolute left-6 md:left-8 lg:left-10 top-0 bottom-0 w-px"
              style={{
                background:
                  "linear-gradient(to bottom, var(--theme-from), transparent 60%)",
                opacity: 0.6,
              }}
            />

            <ul className="space-y-6">
              {items.map((scene, i) => (
                <TimelineRow key={scene.id} scene={scene} index={i} />
              ))}
            </ul>

            {items.length === 0 && (
              <p className="mt-8 text-sm opacity-70 font-mono">No scenes match your filters.</p>
            )}
          </div>
        </div>
      </section>

      <footer className="px-6 md:px-10 lg:px-16 pb-10">
        <div className="mx-auto max-w-6xl text-xs font-mono opacity-60">
          Built in the Icarus & Vega vernacular. Toggle for palette shift; hover edges for ancestry.
        </div>
      </footer>
    </div>
  );
}

function CharacterToggle({
  value,
  onChange,
}: {
  value: Character;
  onChange: (v: Character) => void;
}) {
  return (
    <div
      className="inline-flex rounded-2xl border p-1 bg-black/50"
      style={{ borderColor: "var(--theme-border)" }}
      role="tablist"
      aria-label="Character selector"
    >
      {([
        { key: "icarus", icon: <Flame className="h-4 w-4" />, label: "Icarus" },
        { key: "vega", icon: <Star className="h-4 w-4" />, label: "Vega" },
      ] as const).map((opt) => {
        const active = value === opt.key;
        return (
          <button
            key={opt.key}
            role="tab"
            aria-selected={active}
            onClick={() => onChange(opt.key)}
            className="relative mr-1 last:mr-0 rounded-xl px-3 py-2 text-xs font-mono uppercase tracking-widest"
          >
            <motion.span
              layoutId="character-pill"
              className="absolute inset-0 rounded-xl"
              style={{
                background:
                  active ? "linear-gradient(90deg, var(--theme-from), var(--theme-to))" : "transparent",
                opacity: active ? 0.2 : 0,
              }}
              transition={{ type: "spring", stiffness: 200, damping: 24 }}
            />
            <span className="relative z-10 inline-flex items-center gap-2">
              {opt.icon}
              {opt.label}
            </span>
          </button>
        );
      })}
    </div>
  );
}

function SolarOrb({ character }: { character: Character }) {
  const icon = character === "icarus" ? (
    <Sun className="h-5 w-5" />
  ) : (
    <Star className="h-5 w-5" />
  );
  return (
    <div className="relative h-40 w-40 md:h-48 md:w-48">
      <motion.div
        className="absolute inset-0 rounded-full"
        style={{
          background:
            "radial-gradient(60% 60% at 50% 45%, var(--theme-from) 0%, transparent 70%)",
          filter: "blur(24px)",
          opacity: 0.8,
        }}
        animate={{ scale: [1, 1.03, 1] }}
        transition={{ duration: 6, repeat: Infinity, ease: "easeInOut" }}
      />
      <motion.div
        className="absolute inset-3 rounded-full"
        style={{
          background:
            "conic-gradient(from 30deg, var(--theme-from), var(--theme-to), var(--theme-from))",
          boxShadow: "0 0 24px var(--theme-ring)",
        }}
        animate={{ rotate: 360 }}
        transition={{ duration: 30, repeat: Infinity, ease: "linear" }}
      />
      <div className="absolute inset-6 rounded-full bg-black/60 backdrop-blur-sm border" style={{ borderColor: "var(--theme-border)" }} />
      <div className="absolute inset-0 flex items-center justify-center text-sm" style={{ color: "var(--theme-accent)" }}>
        {icon}
        <span className="ml-2 font-mono tracking-widest uppercase hidden md:inline">
          {character === "vega" ? "Cyan Star" : "Ember Sun"}
        </span>
      </div>
    </div>
  );
}

function LoreCard({
  icon,
  title,
  body,
}: {
  icon: React.ReactNode;
  title: string;
  body: string;
}) {
  return (
    <div className="rounded-2xl border bg-black/40 p-4 md:p-6" style={{ borderColor: "var(--theme-border)" }}>
      <div className="flex items-center gap-2" style={{ color: "var(--theme-accent)" }}>
        {icon}
        <span className="font-mono text-xs tracking-widest uppercase">{title}</span>
      </div>
      <p className="mt-2 text-sm opacity-80">{body}</p>
    </div>
  );
}

function EasterEdges({ children }: { children: React.ReactNode }) {
  return (
    <div className="relative">
      {/* corner glyphs */}
      <motion.span
        className="pointer-events-none absolute -top-2 -left-2 h-10 w-10 rounded-2xl"
        style={{
          background:
            "radial-gradient(closest-side, var(--theme-from), transparent)",
          opacity: 0.2,
        }}
        animate={{ opacity: [0.08, 0.18, 0.08] }}
        transition={{ duration: 8, repeat: Infinity }}
      />
      <motion.span
        className="pointer-events-none absolute -bottom-2 -right-2 h-10 w-10 rounded-2xl"
        style={{
          background:
            "radial-gradient(closest-side, var(--theme-to), transparent)",
          opacity: 0.18,
        }}
        animate={{ opacity: [0.06, 0.16, 0.06] }}
        transition={{ duration: 10, repeat: Infinity }}
      />
      {children}
    </div>
  );
}

function TimelineRow({ scene, index }: { scene: Scene; index: number }) {
  return (
    <motion.li
      initial={{ opacity: 0, x: -10 }}
      animate={{ opacity: 1, x: 0 }}
      transition={{ delay: index * 0.05, duration: 0.35 }}
      className="relative pl-10"
    >
      {/* Node */}
      <div className="absolute left-6 md:left-8 lg:left-10 -translate-x-1/2">
        <div className="relative h-8 w-8">
          <div className="absolute inset-0 rounded-full ring-2" style={{ boxShadow: "0 0 24px var(--theme-ring)", borderColor: "var(--theme-border)" }} />
          <div
            className="absolute inset-2 rounded-full"
            style={{ background: "linear-gradient(135deg, var(--theme-from), var(--theme-to))" }}
          />
        </div>
      </div>

      {/* Card */}
      <div
        className="group rounded-2xl border bg-black/60 backdrop-blur-sm px-4 py-4 md:px-5 md:py-5 transition-colors"
        style={{ borderColor: "var(--theme-border)" }}
      >
        <div className="flex flex-wrap items-center justify-between gap-3">
          <div className="flex items-center gap-3">
            <span
              className="inline-flex items-center rounded-xl px-2 py-1 text-[10px] font-mono uppercase tracking-widest border"
              style={{ borderColor: "var(--theme-border)", background: "var(--theme-tile)" }}
            >
              {scene.id}
            </span>
            <h3 className="text-lg md:text-xl font-bold uppercase tracking-[0.2em]">
              {scene.title}
            </h3>
          </div>

          <div className="flex items-center gap-2">
            <RevealPill value={scene.revealPct} />
            {typeof scene.statusPct === "number" && <StatusBar value={scene.statusPct} />}
          </div>
        </div>

        <div className="mt-3 flex flex-wrap items-center gap-3 text-xs font-mono opacity-80">
          <Meta label="Arc" value={scene.arc} />
          {scene.dependencies?.length ? <Meta label="Deps" value={scene.dependencies.join(", ")} /> : null}
          {scene.tags?.length ? <Meta label="Tags" value={scene.tags.join(" ") } /> : null}
          {scene.date ? <Meta label="Date" value={scene.date} /> : null}
        </div>

        <p className="mt-3 text-sm md:text-base opacity-80">{scene.summary}</p>
      </div>
    </motion.li>
  );
}

function Meta({ label, value }: { label: string; value: string }) {
  return (
    <span
      className="inline-flex items-center gap-2 rounded-xl border px-2 py-1"
      style={{ borderColor: "var(--theme-border)", background: "var(--theme-tile)" }}
    >
      <span className="text-[10px] uppercase tracking-[0.2em] opacity-70">{label}</span>
      <span className="text-xs">{value}</span>
    </span>
  );
}

function RevealPill({ value }: { value: number }) {
  return (
    <span
      title="Reveal percentage"
      className="inline-flex items-center gap-2 rounded-xl border px-2 py-1 font-mono"
      style={{ borderColor: "var(--theme-border)", background: "var(--theme-tile)" }}
    >
      <span className="text-[10px] uppercase tracking-[0.2em] opacity-70">Reveal</span>
      <span className="text-xs">{value}%</span>
    </span>
  );
}

function StatusBar({ value }: { value: number }) {
  const clamped = Math.max(0, Math.min(100, value));
  return (
    <div className="relative h-6 w-40 rounded-xl border overflow-hidden" style={{ borderColor: "var(--theme-border)", background: "rgba(255,255,255,0.04)" }}>
      <div
        className="absolute inset-y-0 left-0"
        style={{
          width: `${clamped}%`,
          background: "linear-gradient(90deg, var(--theme-from), var(--theme-to))",
        }}
      />
      <div className="relative z-10 flex h-full w-full items-center justify-between px-2 text-[10px] font-mono uppercase tracking-widest">
        <span className="opacity-70">Status</span>
        <span>{clamped}%</span>
      </div>
    </div>
  );
}

function Backdrop() {
  return (
    <div aria-hidden className="pointer-events-none fixed inset-0 -z-10">
      {/* theme radial */}
      <div
        className="absolute inset-0"
        style={{
          background:
            "radial-gradient(1000px 600px at 50% -20%, color-mix(in oklab, var(--theme-from) 30%, transparent), transparent)",
        }}
      />
      {/* subtle grid */}
      <div className="absolute inset-0 opacity-[0.06] [background-image:linear-gradient(transparent_95%,rgba(255,255,255,.6)_96%),linear-gradient(90deg,transparent_95%,rgba(255,255,255,.6)_96%)] [background-size:24px_24px]" />
      {/* dust / stars */}
      <div className="absolute inset-0 opacity-20"
        style={{
          backgroundImage:
            "radial-gradient(1px 1px at 10% 20%, white 50%, transparent),radial-gradient(1px 1px at 30% 80%, white 50%, transparent),radial-gradient(1px 1px at 70% 30%, white 50%, transparent),radial-gradient(1px 1px at 90% 60%, white 50%, transparent)",
        }}
      />
    </div>
  );
}
