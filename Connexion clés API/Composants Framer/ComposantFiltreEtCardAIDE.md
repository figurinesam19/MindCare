//  MindCare Ressources — Design Maquette Stricte
//  Couleur Violette et nouveaux filtres (Lignes d'écoute, etc.)
// ============================================================
import { useState, useEffect } from "react"
import { addPropertyControls, ControlType } from "framer"

// ─── CONFIGURATION AIRTABLE ─────────────────────────────────
const AIRTABLE_PAT = "TON_TOKEN_LECTURE_SEULE" 
const AIRTABLE_BASE_ID = "TA_BASE_ID"
const AIRTABLE_TABLE = "Ressources" 
// ────────────────────────────────────────────────────────────

// Nouveaux filtres basés sur ta capture d'écran
const FILTERS = [
    { label: "Toutes", icon: "" },
    { label: "Lignes d'écoute", icon: "📞" },
    { label: "Associations", icon: "🤝" },
    { label: "Centre d'accueil", icon: "🏠" }
]

export default function ResourceCenter({ accentColor, activeFilterColor }) {
    const [resources, setResources] = useState([])
    const [activeFilter, setActiveFilter] = useState("Toutes")
    const [status, setStatus] = useState("loading")

    // Le violet redevient la couleur dominante
    const PURPLE = activeFilterColor || "#7B2D8B" 
    const BADGE_COLOR = accentColor || "#D8A03E" // Optionnel : couleur pour l'étiquette statut
    const BG_COLOR = "#F6F5F0" 
    const TEXT = "#1A1A1A"
    const MUTED = "#7A7A7A"

    useEffect(() => {
        fetchResources()
    }, [])

    const fetchResources = async () => {
        try {
            const res = await fetch(
                `https://api.airtable.com/v0/${AIRTABLE_BASE_ID}/${encodeURIComponent(AIRTABLE_TABLE)}`,
                {
                    headers: {
                        Authorization: `Bearer ${AIRTABLE_PAT}`,
                    },
                }
            )
            if (!res.ok) throw new Error("Erreur de chargement")
            const data = await res.json()
            
            const formattedData = data.records.map(record => ({
                id: record.id,
                titre: record.fields.titre || "Ressource",
                typeContenu: record.fields["Type de contenu"] || "Associations", 
                statut: record.fields.statut || "", 
                description: record.fields.description || "",
                sourceURL: record.fields.sourceURL || "",
                lieu: record.fields.Lieu || "En ligne",
                horaires: record.fields.Horaires || "Toujours disponible",
                tags: ["Aide", "Gratuit"] 
            }))
            
            setResources(formattedData)
            setStatus("success")
        } catch (error) {
            console.error(error)
            setStatus("error")
        }
    }

    const filteredResources = activeFilter === "Toutes" 
        ? resources 
        : resources.filter(r => r.typeContenu === activeFilter)

    const s = {
        container: { padding: "20px", fontFamily: "'DM Sans', sans-serif", width: "100%", maxWidth: 500, margin: "0 auto", background: BG_COLOR, minHeight: "100vh" },
        
        filtersContainer: { background: "#FFF", borderRadius: "16px", padding: "16px", marginBottom: "24px", boxShadow: "0px 2px 10px rgba(0,0,0,0.02)" },
        filtersWrap: { display: "flex", flexWrap: "wrap", gap: "10px" },
        filterChip: (isActive) => ({
            padding: "8px 16px",
            borderRadius: "20px",
            fontSize: "14px",
            fontWeight: 600,
            cursor: "pointer",
            background: isActive ? PURPLE : "#F5F5F5",
            color: isActive ? "#FFF" : "#555", // Texte blanc sur fond violet
            border: isActive ? `1px solid ${PURPLE}` : "1px solid #EAEAEA",
            display: "flex",
            alignItems: "center",
            gap: "6px",
            transition: "all 0.2s ease"
        }),

        card: { background: "#FFF", borderRadius: "16px", padding: "24px", marginBottom: "16px", boxShadow: "0px 2px 10px rgba(0,0,0,0.02)" },
        cardHeader: { display: "flex", alignItems: "center", gap: "10px", marginBottom: "16px", flexWrap: "wrap" },
        
        typeBadge: { 
            background: `${PURPLE}15`, // Fond violet très clair (transparence)
            color: PURPLE, // Texte violet
            padding: "6px 14px", 
            borderRadius: "20px", 
            fontSize: "11px", 
            fontWeight: 800,
            textTransform: "uppercase",
            letterSpacing: "0.5px"
        },
        
        badgeStatus: { background: BADGE_COLOR, color: "#FFF", padding: "6px 14px", borderRadius: "20px", fontSize: "11px", fontWeight: 700 },
        
        title: { fontSize: "18px", fontWeight: 700, color: TEXT, margin: "0 0 12px 0", lineHeight: 1.3 },
        description: { fontSize: "14px", color: MUTED, lineHeight: 1.5, margin: "0 0 20px 0" },
        tagsWrap: { display: "flex", flexWrap: "wrap", gap: "8px", marginBottom: "24px" },
        tagGray: { background: "#F5F5F5", color: "#555", padding: "8px 14px", borderRadius: "16px", fontSize: "12px", fontWeight: 500 },
        infoList: { display: "flex", flexDirection: "column", gap: "12px", borderTop: "1px solid #EAEAEA", paddingTop: "16px" },
        infoItem: { display: "flex", alignItems: "center", gap: "8px", fontSize: "13px", color: MUTED },
        link: { color: PURPLE, textDecoration: "none", fontWeight: 600, display: "flex", alignItems: "center", gap: "6px" }
    }

    if (status === "loading") return <div style={{...s.container, textAlign: "center", color: MUTED, paddingTop: 40}}>Chargement...</div>
    if (status === "error") return <div style={{...s.container, textAlign: "center", color: "red", paddingTop: 40}}>Erreur Airtable.</div>

    return (
        <div style={s.container}>
            <div style={s.filtersContainer}>
                <div style={s.filtersWrap}>
                    {FILTERS.map(filter => (
                        <div 
                            key={filter.label} 
                            style={s.filterChip(activeFilter === filter.label)}
                            onClick={() => setActiveFilter(filter.label)}
                        >
                            {filter.icon && <span>{filter.icon}</span>}
                            {filter.label}
                        </div>
                    ))}
                </div>
            </div>

            <div>
                {filteredResources.map(res => (
                    <div key={res.id} style={s.card}>
                        <div style={s.cardHeader}>
                            <span style={s.typeBadge}>{res.typeContenu}</span>
                            {res.statut && !res.statut.toLowerCase().includes("publier") && (
                                <span style={s.badgeStatus}>{res.statut}</span>
                            )}
                        </div>
                        
                        <h3 style={s.title}>{res.titre}</h3>
                        <p style={s.description}>{res.description}</p>
                        
                        <div style={s.tagsWrap}>
                            {res.tags.map((tag, i) => (
                                <span key={i} style={s.tagGray}>{tag}</span>
                            ))}
                        </div>
                        
                        <div style={s.infoList}>
                            <div style={s.infoItem}>📍 {res.lieu}</div>
                            <div style={s.infoItem}>🕒 {res.horaires}</div>
                            {res.sourceURL && (
                                <div style={s.infoItem}>
                                    <a href={res.sourceURL} target="_blank" rel="noopener noreferrer" style={s.link}>
                                        ↗ Visiter le site web
                                    </a>
                                </div>
                            )}
                        </div>
                    </div>
                ))}
            </div>
        </div>
    )
}

addPropertyControls(ResourceCenter, {
    activeFilterColor: { type: ControlType.Color, title: "Couleur Dominante (Violet)", defaultValue: "#7B2D8B" },
    accentColor: { type: ControlType.Color, title: "Couleur Badge Statut", defaultValue: "#D8A03E" },
})