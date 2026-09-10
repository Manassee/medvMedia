# MEDV Media
Die MEDV Mediathek ist die offizielle Medien-App der Gemeinde MEDV. Sie verbindet das Beste aus YouTube (Predigten, Gottesdienste, Livestream) und Instagram (Reels, Fotogalerien) in einer App, die ohne Anmeldung für alle öffentlich nutzbar ist.

Inhalte werden ausschließlich vom Mediateam veröffentlicht. Zuschauer können Inhalte ansehen, liken, als Favorit speichern, teilen und offline herunterladen

## Architektur

```mermaid
flowchart TB
    subgraph gemeinde["Gemeinde"]
        vmix["vMix"]
    end

    yt["YouTube"]
    fb["Facebook"]

    subgraph hetzner["Hetzner Server (Docker)"]
        proxy["Reverse Proxy (TLS)"]
        mtx["MediaMTX<br/>Live → HLS, Aufzeichnung"]
        api["ASP.NET Core API<br/>.NET 10"]
        admin["Blazor Admin-Panel"]
        jobs["Hangfire<br/>Hintergrundjobs"]
        db[("PostgreSQL")]
    end

    subgraph bunny["Bunny.net"]
        pull["Pull Zone<br/>Livestream-CDN"]
        stream["Bunny Stream<br/>Videos, Reels, Untertitel"]
        storage["Bunny Storage + CDN<br/>Fotos, Audios"]
    end

    app["Expo App<br/>iOS & Android"]
    push["Expo Push<br/>APNs / FCM"]

    vmix -- RTMP --> yt
    vmix -- RTMP --> fb
    vmix -- "SRT / RTMP (720p)" --> mtx
    mtx -- "Webhook: Stream bereit" --> api
    mtx -- "Origin Live-HLS" --> pull
    api <--> db
    jobs <--> db
    admin --> api
    admin -- "Upload (TUS)" --> stream
    jobs -- "Aufzeichnung hochladen" --> stream
    jobs -- "Fotos & Audios" --> storage
    app -- REST --> proxy --> api
    pull --> app
    stream --> app
    storage --> app
    api --> push --> app
```
