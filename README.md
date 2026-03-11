# Progetto Infrastruttura IT e Sicurezza - Theta Company

## 📖 Descrizione del Progetto
Questo repository contiene il progetto finale per la messa in sicurezza e l'adeguamento infrastrutturale della Theta Company. L'infrastruttura è stata riprogettata per supportare in modo efficiente 120 utenti distribuiti su 6 piani, adottando un rigoroso approccio di sicurezza basato sulla "Defense in Depth" (Difesa in Profondità). 

Il lavoro copre l'intero ciclo di vita del progetto: dal design della rete gerarchica e segmentata, alla configurazione avanzata dei firewall per la gestione delle zone di sicurezza (LAN, DMZ, WAN), fino allo sviluppo di tool di sicurezza proprietari in Python per condurre audit interni senza dipendere da software di terze parti.

## 📂 Struttura della Repository
Il progetto è suddiviso in tre directory principali per facilitarne la consultazione:

* **`PY/` (Script Custom):** Contiene i tool di sicurezza sviluppati ad-hoc in Python per l'auditing della rete.
* **`pt/` (Simulazione Rete):** Contiene il file Cisco Packet Tracer con la topologia fisica e logica dell'infrastruttura.
* **`rel/` (Documentazione):** Raccoglie i report tecnici dettagliati, le analisi del codice e la giustificazione del preventivo economico.

---

## 🛠️ Dettaglio delle Componenti

### 1. Architettura e Sicurezza di Rete (`pt/` e `rel/`)
L'infrastruttura simulata risponde a standard Enterprise abbandonando logiche permissive:



* **Segmentazione Logica:** Implementazione di VLAN dedicate per ogni piano e per i servizi critici, con assegnazione dinamica degli IP gestita da un Server DHCP centralizzato.
* **Doppia Barriera e DMZ:** Utilizzo di un Firewall Perimetrale (con NAT/PAT) e un Firewall Interno per segmentare in modo netto la rete fidata (Inside), Internet (Outside) e la Zona Demilitarizzata (DMZ).
* **Controllo Accessi (pfSense):** Regole stringenti che applicano il Principio del Privilegio Minimo: solo la postazione IT Admin può gestire i server in SSH, mentre la rete aziendale ha accesso esclusivo alla porta HTTP (80) della DMZ. Il traffico laterale non autorizzato è bloccato (Reject).
* **Protezione Dati (IDS/IPS):** Il NAS aziendale è isolato e protetto, mentre 3 sistemi Sniffer (IPS-EXT, IPS-NAS, IDS) monitorano costantemente il traffico nei punti nevralgici.
* **Budget:** Analisi tecnico-economica che giustifica un investimento complessivo di € 226.000 per l'adeguamento hardware e software.

### 2. Strumenti Python per l'Auditing (`PY/`)
Come richiesto dalle policy di progetto, sono stati creati script custom per mappare la superficie di attacco, vietando l'uso di tool come Nmap.

* **`portscnRange.py` e `portscanALL.py`:** Scanner TCP proprietari. Evitano approcci bloccanti utilizzando `connect_ex` e timeout per discriminare lo stato delle porte (Aperta, Chiusa, Filtrata) basandosi sui codici di errore del sistema operativo (es. `111 ECONNREFUSED` o `110 ETIMEDOUT`). I risultati vengono salvati in un file di log.
* **`sockPktscn.py`:** Un "packet sniffer" raw che opera a Livello 2 (OSI) bypassando lo stack TCP/IP del sistema. Cattura i frame, decodifica l'header IPv4 e filtra il traffico in base a IP sorgente o destinazione.
* **`httpVerbScan.py`:** Strumento di ricognizione web automatizzata che sfrutta la libreria `requests`. Interroga un target specifico inviando richieste (`OPTIONS`, `GET`, `POST`, `PUT`, `DELETE`) per verificare quali metodi HTTP siano abilitati e gestiti dal server, leggendo gli header `Allow` e `Server`.

---

## 💻 Istruzioni per l'Home Lab (VirtualBox)

Il codice Python in `PY/` è progettato per essere testato nel nostro ambiente di rete locale isolato (`kalinet` - `192.168.100.0/24`). 

Per testare gli script, accedi alla **Kali Linux VM (Attaccante - 192.168.100.10)** e utilizza la **Metasploitable VM (Target - 192.168.100.11)** come bersaglio.



### Esecuzione degli Script

1.  **Port Scanning (Scansione Servizi):**
    ```bash
    python3 PY/portscnRange.py
    ```
    * **Cosa fa:** Verifica quali porte TCP sono in ascolto sul bersaglio.
    * **Input consigliato:** Target IP `192.168.100.11`, Range `20-100`.
    * **Expected Output:** Mostrerà a terminale le porte aperte (es. `Port 80: OPEN`) e genererà un file log dettagliato.

2.  **Packet Sniffing (Analisi Traffico):**
    ```bash
    sudo python3 PY/sockPktscn.py
    ```
    * **Cosa fa:** Intercetta i pacchetti di rete a basso livello diretti o provenienti dall'IP specificato.
    * **Nota:** *I raw sockets richiedono privilegi di root, per questo usiamo `sudo`*.
    * **Input consigliato:** Inserisci l'IP target (es. `192.168.100.11`). Puoi aprire un altro terminale e fare un `ping 192.168.100.11` per generare traffico da catturare.

3.  **HTTP Verb Verifier (Ricognizione Web):**
    ```bash
    python3 PY/httpVerbScan.py
    ```
    * **Cosa fa:** Testa l'abilitazione di metodi potenzialmente pericolosi come `PUT` o `DELETE` sul server web.
    * **Input consigliato:** Base URL `http://192.168.100.11/`, Path `/` o `/dvwa/`.
    * **Expected Output:** Una tabella che mostra il codice di stato HTTP restituito per ogni metodo testato e un'analisi sui metodi verosimilmente abilitati.
