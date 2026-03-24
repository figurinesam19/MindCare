import { useState, useEffect } from "react"
import { addPropertyControls, ControlType } from "framer"

// Configuration (À remplacer par vos vrais identifiants)
const CLIENT_ID = "VOTRE_CLIENT_ID"
const CLIENT_SECRET = "VOTRE_CLIENT_SECRET"

export default function MentalHealthPodcasts(props) {
    const { searchTerm, limit } = props
    const [podcasts, setPodcasts] = useState([])
    const [loading, setLoading] = useState(true)

    useEffect(() => {
        async function getSpotifyData() {
            try {
                // 1. Obtenir le Token d'accès (Obligatoire à chaque fois)
                const authResponse = await fetch(
                    "https://accounts.spotify.com/api/token",
                    {
                        method: "POST",
                        headers: {
                            "Content-Type": "application/x-www-form-urlencoded",
                            Authorization:
                                "Basic " +
                                btoa(CLIENT_ID + ":" + CLIENT_SECRET),
                        },
                        body: "grant_type=client_credentials",
                    }
                )
                const authData = await authResponse.json()
                const token = authData.access_token

                // 2. Rechercher les podcasts sur la santé mentale / bien-être
                // On utilise "type=show" pour les podcasts
                const searchResponse = await fetch(
                    `https://api.spotify.com/v1/search?q=${encodeURIComponent(searchTerm)}&type=show&market=FR&limit=${limit}`,
                    {
                        headers: { Authorization: `Bearer ${token}` },
                    }
                )
                const searchData = await searchResponse.json()
                setPodcasts(searchData.shows.items)
                setLoading(false)
            } catch (error) {
                console.error("Erreur Spotify:", error)
                setLoading(false)
            }
        }

        getSpotifyData()
    }, [searchTerm, limit])

    if (loading)
        return <div style={centerStyle}>Chargement des podcasts...</div>

    return (
        <div style={containerStyle}>
            {podcasts.map((podcast) => (
                <a
                    key={podcast.id}
                    href={podcast.external_urls.spotify}
                    target="_blank"
                    style={cardStyle}
                >
                    <img
                        src={podcast.images[0]?.url}
                        style={imageStyle}
                        alt={podcast.name}
                    />
                    <div style={textContainer}>
                        <h4 style={titleStyle}>{podcast.name}</h4>
                        <p style={publisherStyle}>{podcast.publisher}</p>
                    </div>
                </a>
            ))}
        </div>
    )
}

// --- Styles et Contrôles Framer ---

const containerStyle = {
    display: "flex",
    flexDirection: "column",
    gap: "12px",
    padding: "10px",
    overflowY: "auto",
}
const cardStyle = {
    display: "flex",
    gap: "15px",
    alignItems: "center",
    textDecoration: "none",
    color: "inherit",
    background: "rgba(0,0,0,0.05)",
    borderRadius: "12px",
    padding: "10px",
}
const imageStyle = {
    width: "60px",
    height: "60px",
    borderRadius: "8px",
    objectFit: "cover",
}
const textContainer = { display: "flex", flexDirection: "column", gap: "4px" }
const titleStyle = { margin: 0, fontSize: "14px", fontWeight: "bold" }
const publisherStyle = { margin: 0, fontSize: "12px", opacity: 0.7 }
const centerStyle = {
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
    height: "100%",
}

import { useState, useEffect } from "react"
import { addPropertyControls, ControlType } from "framer"

// Configuration (À remplacer par vos vrais identifiants)
const CLIENT_ID = "VOTRE_CLIENT_ID"
const CLIENT_SECRET = "VOTRE_CLIENT_SECRET"

export default function MentalHealthPodcasts(props) {
    const { searchTerm, limit } = props
    const [podcasts, setPodcasts] = useState([])
    const [loading, setLoading] = useState(true)

    useEffect(() => {
        async function getSpotifyData() {
            try {
                // 1. Obtenir le Token d'accès (Obligatoire à chaque fois)
                const authResponse = await fetch(
                    "https://accounts.spotify.com/api/token",
                    {
                        method: "POST",
                        headers: {
                            "Content-Type": "application/x-www-form-urlencoded",
                            Authorization:
                                "Basic " +
                                btoa(CLIENT_ID + ":" + CLIENT_SECRET),
                        },
                        body: "grant_type=client_credentials",
                    }
                )
                const authData = await authResponse.json()
                const token = authData.access_token

                // 2. Rechercher les podcasts sur la santé mentale / bien-être
                // On utilise "type=show" pour les podcasts
                const searchResponse = await fetch(
                    `https://api.spotify.com/v1/search?q=${encodeURIComponent(searchTerm)}&type=show&market=FR&limit=${limit}`,
                    {
                        headers: { Authorization: `Bearer ${token}` },
                    }
                )
                const searchData = await searchResponse.json()
                setPodcasts(searchData.shows.items)
                setLoading(false)
            } catch (error) {
                console.error("Erreur Spotify:", error)
                setLoading(false)
            }
        }

        getSpotifyData()
    }, [searchTerm, limit])

    if (loading)
        return <div style={centerStyle}>Chargement des podcasts...</div>

    return (
        <div style={containerStyle}>
            {podcasts.map((podcast) => (
                <a
                    key={podcast.id}
                    href={podcast.external_urls.spotify}
                    target="_blank"
                    style={cardStyle}
                >
                    <img
                        src={podcast.images[0]?.url}
                        style={imageStyle}
                        alt={podcast.name}
                    />
                    <div style={textContainer}>
                        <h4 style={titleStyle}>{podcast.name}</h4>
                        <p style={publisherStyle}>{podcast.publisher}</p>
                    </div>
                </a>
            ))}
        </div>
    )
}

// --- Styles et Contrôles Framer ---

const containerStyle = {
    display: "flex",
    flexDirection: "column",
    gap: "12px",
    padding: "10px",
    overflowY: "auto",
}
const cardStyle = {
    display: "flex",
    gap: "15px",
    alignItems: "center",
    textDecoration: "none",
    color: "inherit",
    background: "rgba(0,0,0,0.05)",
    borderRadius: "12px",
    padding: "10px",
}
const imageStyle = {
    width: "60px",
    height: "60px",
    borderRadius: "8px",
    objectFit: "cover",
}
const textContainer = { display: "flex", flexDirection: "column", gap: "4px" }
const titleStyle = { margin: 0, fontSize: "14px", fontWeight: "bold" }
const publisherStyle = { margin: 0, fontSize: "12px", opacity: 0.7 }
const centerStyle = {
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
    height: "100%",
}

addPropertyControls(MentalHealthPodcasts, {
    searchTerm: {
        type: ControlType.String,
        title: "Recherche",
        defaultValue: "santé mentale bien-être",
    },
    limit: {
        type: ControlType.Number,
        title: "Nombre",
        min: 1,
        max: 20,
        defaultValue: 5,
    },
})
