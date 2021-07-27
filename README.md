# Die Cloud Native Architektur
Ausarbeitung für das Pflichtmodul Software-Architektur (PIM-SA) an der Hochschule für Technik und Wirtschaft des 
Saarlandes im Studiengang Praktische Informatik der Fakultät für Ingenieurwissenschaften


#### von Nicolas Klein, Niklas Schon, Julian Krieger

#### betreut von

#### Prof. Dr. Markus Esch

#### Saarbrücken, 22. März 2021


## Inhaltsverzeichnis


- 1 Einleitung
   - 1.1 Motivation
   - 1.2 Struktur der Arbeit
- 2 Grundlagen
   - 2.1 Die Cloud
      - 2.1.1 Service Modelle
      - 2.1.2 Deployment Modelle
      - 2.1.3 Vorteile und Möglichkeiten der Cloud
   - 2.2 Anforderungen
   - 2.3 Prinzipien einer Cloud Native Architektur
      - 2.3.1 Automatisierung
      - 2.3.2 Sicherheit
      - 2.3.3 Sinnvoller Einsatz von State
   - 2.4 Cloud-Native-Technologien
      - 2.4.1 Micro-Services-Architektur
      - 2.4.2 Container
      - 2.4.3 Service-Meshes
- 3 Analyse
   - 3.1 Differenzierung
   - 3.2 Zusammenfassung
- 4 Tools und Industriestandards
   - 4.1 Container Runtime Environments
      - 4.1.1 Container Runtime
      - 4.1.2 Linux Control Groups (cgroups)
      - 4.1.3 Linux Containers (LXC / LXD)
      - 4.1.4 LMCTFY (Let Me Contain That For You)
      - 4.1.5 Docker
      - 4.1.6 rkt („Rocket“)
      - 4.1.7 Podman
      - 4.1.8 Open Container Initiative Runtimes
      - 4.1.9 Kubernetes Container Runtime Interface (CRI)
   - 4.2 Container Orchestrator
      - 4.2.1 Kubernetes
      - 4.2.2 Docker Swarm
      - 4.2.3 Docker Compose
      - 4.2.4 Fleet
      - 4.2.5 Apache Mesos
   - 4.3 Cloud-Anbieter und Managed Kubernetes Services
   - 4.4 Nutzung in der Industrie
- 5 Fallstudie
   - 5.1 Vorstellung der Fallstudie
   - 5.2 Anforderungen
      - 5.2.1 Funktionale Anforderungen
      - 5.2.2 Nichtfunktionale Anforderungen
   - 5.3 Konzeption
   - 5.4 Umsetzung
   - 5.5 Evaluation
      - 5.5.1 Umgang mit State
      - 5.5.2 Die Datenbank
      - 5.5.3 Kommunikations-Logik
   - 5.6 Fazit
- Literatur
- Abbildungsverzeichnis
- Tabellenverzeichnis
- Listings


## 1 Einleitung

### 1.1 Motivation


Ob Film-Verleih, Taxi-Geschäft oder Buchhandel: All diese Wirtschaftszweige wurden im
letzten Jahrzehnt vollständig transformiert. Dies ist nicht zuletzt auf die Fortschritte in
der Software-Entwicklung zurück zu führen. Dass diese für neue Geschäftsmodelle von
Bedeutung ist, bemerkte Watts S. Humphrey bereits im Jahr 2002. In seinem BuchWinning
with Software: An Executive Strategyscheibt er:

```
”Current trends suggest that, regardless of the industry you are in, your future
products will use more software and be more complex than those of today.”
[24]
```

Scholl et Al. bezeichnen solche Firmen als ”Born-In-The-Cloud”[44].
Der BegriffCloudstammt dabei von dem BegriffCloud Computingab. Die National Institu-
te of Standards and Technology definiert Cloud Computing als ”[...] on-demand network
access to a shared pool of configurable computing resources [...] that can be rapidly provi-
sioned and released with minimal management [...]”[32].
In der Praxis bedeutet dies, dass die benötigten Server-Ressourcen nicht von Firmen selbst
verwaltet werden müssen.
Dies ermöglicht es Firmen den Aspekt der Server-Verwaltung an Dritte auszulagern. Der
Fokus kann daher auf das eigene Geschäft und weniger auf die konkrete Infrastruktur
gelegt werden. Rechenleistung wird dabei konkret in einem proportionalen Verhältnis zu
den aktuellen Bedürfnissen eingekauft.
Damit ist es Firmen möglich, ihre Angebote flexibel an wachsende Kunden-zahlen an-
zupassen. Applikationen, welche diese Geschäftsmodelle nutzen und die Möglichkeiten
der Cloud ausschöpfen, werden alsCloud-Native Applikationenbezeichnet [44]. Sicherheit,
Skalierbarkeit und Erweiterbarkeit sind nur ein Teil der Anforderungen, welche bei der
Entwicklung einer Cloud-Native Architektur betrachtet werden müssen.

### 1.2 Struktur der Arbeit

Diese Arbeit ist wie folgt strukturiert. Zunächst wird im ersten Kapitel der Begriff Cloud-
Computing näher erläutert. Dabei werden Anforderungen an eine Cloud-Architektur
erarbeitet. Weiterhin wird gezeigt wie Cloud-Native Technologien diese lösen können.
Kapitel zwei analysiert die Cloud-Native Architektur anhand ihrer Vor -und Nachteile.
In Kapitel drei wird auf konkrete Technologien für die Umsetzung einer Cloud-Native
Architektur eingegangen. Schließlich wird in Kapitel vier eine Cloud-Native Fallstudie
vorgestellt und evaluiert.



## 2 Grundlagen


Um die Cloud-Native-Architektur zu betrachten, gilt es erst einige Grundlagen zu erarbei-
ten. In diesem Kapitel wird zunächst der Begriff Cloud abgegrenzt. Dabei wird auch der
Unterschied vonPublicundPrivate Cloudbetrachtet. Daraus wird anschließend eine Liste
von Anforderungen an eine Cloud-Architektur formuliert. Aufbauend auf dieser wird
abschließend die Cloud Native Architektur definiert und deren Eigenschaften betrachtet.

### 2.1 Die Cloud

Sofern eine Software in Benutzung ist, muss sie ständig angepasst und verbessert werden.
Dies zeigte sich bei der Einführung der europäischen Datenschutz Grundverordnung
(DSGVO) im Jahr 2018. Letztere stellte unter Anderem neue Anforderung an den Um-
gang mit personenbezogenen Daten. Bestehende Software-Systeme müssen sich diesen
Anforderungen stellen.
Wegen nicht-beachtung dieser Anforderungen musste der Bekleidungshersteller H&M
im Jahr 2020 ein Bußgeld von 35,3 Millionen Euro zahlen. Um auf äußere Einflüsse zu
Reagieren, muss eine Software folglich stets flexibel bleiben.
Allerdings widerstehen über viele Jahre gewachsene Software-Systeme meist Änderun-
gen. Neue Anforderungen haben zu häufigen Modifikationen dieser Systeme geführt, was
wiederum zu unstrukturiertem Quellcode führt, der schwer zu pflegen und anzupassen
ist. Darüber hinaus wird das Wissen über die Altsysteme knapp, da die ursprünglichen
Programmierer das Unternehmen verlassen oder in den Ruhestand gehen. Ferner fehlt es
in der Regel an Dokumentation.
Eine Pflege des Systems ist daher von Nöten. Softwaremodernisierung ist "[...] der Prozess
der Weiterentwicklung bestehender Softwaresysteme durch Ersetzung, Neuentwicklung,

#### 2.1.1 Service Modelle

Verschiedene Geschäftsmodelle des Cloud-Computings setzen unterschiedliche Grade
der Abstraktion um. Hierbei wird zwischen den Service ModellenInfrastructure as a Ser-
vice,Plattform as a ServiceundSoftware as a Serviceunterschieden. Im Nachfolgenden
werden diese Modelle voneinander abgegrenzt.

**2.1.1.1 Infrastructure as a Service**
Das sogenannteInfrastructure as a Service (IaaS)[32] stellt Speicher -und Rechenleistung
zur Verfügung. Der Kunde muss also nicht selbst physische Hardware betreiben. IaaS
kann als Grundschicht unter allen anderen Service Modellen gesehen werden.
Mittels Virtualisierung teilt der Service-Provider die physische Hardware auf und teilt sie
den Kunden zu. Der Kunde hat also kein Einfluss auf die zugrundeliegende Infrastruktur.
Allerdings kann er Aspekte wie Speicher, Betriebssystem, Middle-Ware und die Anwen-
dung selbst konfigurieren [13].
Auch ein begrenzter Zugriff auf Netzwerkomponenten, wie zum Beispiel die Firewall,
kann möglich sein. IaaS stellt demnach die selben Funktionen wie ein traditionelles


Rechen-Center zur Verfügung. Der Kunde muss allerdings kein solches Instandhalten
und kann flexibel die Rechenleistung erhöhen oder verringern.
Abgerechnet wird in der Regel entsprechend der Nutzungsdauer. In einigen Fällen über-
nimmt der Service-Provider auch Aufgaben wie die Systemwartung, Datensicherung und
das Notfall-Management.
Unter anderem sind Microsoft, IBM und Amazon Anbieter von IaaS Angeboten. [45] [32]

**2.1.1.2 Plattform as a Service**

DasPlattform as a Service (PaaS)[32] Model abstrahiert gegenüber IaaS weitere Aspekte
der Cloud. Weiterhin muss der Kunde keine eigene Rechenzentren verwalten und erhält
durch den Service Provider flexibel Rechenleistung.
Allerdings werden nun auch Aspekte wie das Betriebssystem, die Speicherverwaltung
mit Datenbanken und das Netzwerk, durch den Service Provider verwaltet. Der Kunde
erhält eine Entwicklungsumgebung in der Cloud, über welche der die Anwendung be-
reitstellen kann.
PaaS ist dafür konzipiert den kompletten Lebenszyklus einer Anwendung zu ermögli-
chen. Hierzu zählen das bauen, testen, veröffentlichen, verwalten und aktualisieren einer
Anwendung [13]. Auf einer höheren Abstraktion kommen neben Entwicklungswerkzeu-
gen, Programmiersprachen, Bibliotheken und Datenbanken auch Container-Techniken
dazu.
Zu den wichtigsten PaaS Anbietern gehören Amazon, IBM und Microsoft. [46][47] [32]

**2.1.1.3 Software as a Service**

Software as a Service (SaaS)ist die am meisten abstrahierte Form des Cloud-Computings.
Kunden können über das Internet auf Angebote zugreifen, welche von einem Service-
Provider zur Verfügung gestellt werden. Die Applikation wird durch den Service-
Provider verwaltet, betrieben und aktualisiert.
Der Nutzer verwaltet keinen Aspekte der zugrundeliegenden Infrastruktur. Einzig eine
konfigurieren der Anwendung selbst kann möglich sein.
Typische SaaS-Applikationen im Business Bereich sindGoogle G Suite[23] undMicrosoft
Office 365[38].
SaaS-Provider rechnen Anwendungen in der Regen anhand bestimmter Parameter, wie
die Anzahl der Nutzer, ab. Auch bei SaaS-Angeboten können Kunden bei Bedarf einzelne
Dienste oder Funktionen stärker in Anspruch nehmen. [50] [47] [32]

#### 2.1.2 Deployment Modelle

Neben dem Grad der Abstraktion stellt auch der Ort der Hardware eine Rolle. Wird
diese lokal von dem eigenen Unternehmen verwaltet? Oder ist eine externe Firma da-
für verantwortlich? Hierfür existieren einige Modell, welche im Folgenden voneinander
differenziert werden:

**on premise**
Auch als on-premise Cloud oder private Cloud bezeichnet [13]. Die Infrastruktur
wird exklusiv von einer Organisation genutzt [32]. Die Verwaltung der Infrastruktur
kann ebenfalls durch diese Organisation erfolgen. Alternativ kann eine externe
Firma damit beauftragt werden. Die benötigte Hardware befindet sich entweder
in-house (on-premise) oder oder wirdoff-premiseextern bereitgestellt [51].



2.1 Die Cloud

**public Cloud**
Die Infrastruktur wird durch einen Dritt-Anbieter über das Internet zur Verfügung
gestellt. Die gesamte Hardware ist im Besitzt des Service-Providers. Die selbe Hard-
ware, Speicher und Netzwerk-Geräte des Service-Providers werden mit anderen
Kunden geteilt. Beispiele hierfür wären Microsoft Azure oder Amazon Web Ser-
vices. [34] [32]

**hybrid Cloud**
Hybrid Clouds bestehen aus mehreren getrennten Cloud-Infrastrukturen. Daten
und Applikationen einer hybrid Cloud können zwischen private Cloud und public
Cloud Infrastrukturen wechseln. [32]

#### 2.1.3 Vorteile und Möglichkeiten der Cloud

Jedes der oben genannten Deployment Modelle und Service Modelle hat eine Menge
von Eigenschaften gemeinsam. Avram et al. nennt folgende Eigenschaften: (i) pay-per-
use (kein fortlaufenden Verpflichtungen), (ii) elastic capacity and the illusion of infinite
resources; (iii) self-service-interface; und (iv) resources that are abstracted or virtualized
[3]. Aus diesen folgend direkt einige Vorteile des Cloud-Computings.

- Durch das pay-per-use Modell muss der Kunde nur für die Leistung zahlen, die
    er tatsächlich benötigt. Dies reduziert Hardware-Kosten, wie auch die Kosten für
    Verwaltung, Software-Updates und den Strom. Ferner wird weniger Personal für
    die Verwaltung der IT benötigt. Dies ermöglicht auch kleineren Firmen rechen-
    aufwändige Analysen zu bewältigen. Letztere waren zuvor nur größeren Firmen
    vorbehalten. Diese umfangreichen Berechnungen benötigen meist viel Leistung, für
    einen kurzen Zeitraum. Durch das self-service-interface (iii) und die dynamische
    Kapazität von Leistung (iv) mit pay-per-use (i) ist dies möglich. Avram et al [3]
    sieht hier zusätzlich eine Möglichkeit für Dritte-Welt-Länder zu dem Westen aufzu-
    schließen. Länder welche traditionell nicht die benötigen Ressourcen gehabt hätten,
    können nun diese durch Cloud-Computing beziehen.
- Durch das Cloud-Computing können neue Business-Modell umgesetzt werden. Die
    Cloud ermöglicht einen direkten Zugriff auf Rechenleistung, ohne eine Vorherige
    Kapital-Anlage des Kunden [39]. Es muss nicht zuvor Geld in ein Rechenzentrum
    oder zusätzliche IT-Experten investiert werden. Die Zeit bis ein neues Produkt ge-
    winnbringend an den Markt gebracht werden kann, ist demnach deutlich gesunken.
    Viele Internet-Startups konnten dadurch gegründet und zum Erfolg geführt werden
    [3].
- Cloud-Computing erleichtert es Firmen, ihre Services zu skalieren. Wächst die Zahl
    der Nutzer, kann Rechenleistung dazu gekauft werden. Die Kosten steigen daher
    dynamisch mit der Reichweite des Unternehmens. Auch eine Reaktion auf ein kurz-
    fristigen Anstieg oder Abfall der Nutzer-Anfragen ist möglich. Eines der Ziele des
    Cloud-Computings besteht darin, Ressourcen mithilfe von Software-APIs je nach
    Client-Auslastung bei minimaler Interaktion mit dem Service-Provider dynamisch
    zu vergrößern oder zu verkleinern [3]. Dies ist möglich, da die Rechen-Ressourcen
    durch Software verwaltet werden. Für eine dynamische Skalierung, wird zusätz-
    lich eine passende Software-Architektur benötigt. Hierauf wird in einem späteren
    Kapitel eingegangen.
- Hier ist außerdem der Gedanke aufzugreifen, dass eine Fokussierung auf das Ge-
    schäftsmodell möglich wird. Firmen können sich auf ihr eigentliches Geschäftsmo-


2 Grundlagen


dell konzentrieren. Zeit und Geld können in das Geschäftsmodell und nicht in die
IT-Infrastruktur investiert werden.

### 2.2 Anforderungen

Nachdem wir im letzten Abschnitt den Begriff Cloud-Computing erläutert haben, gilt es
nun Cloud-Applikationen zu betrachten. Die meisten der zuvor genannten Vorteile des
Cloud-Computings können nur durch eine passende Software-Architektur vollkommen
ausgeschöpft werden. Beim Entwurf der Architektur einer Cloud-Applikation müssen
einige Aspekte beachtet werden. Folgende Anforderungen haben sich dabei herauskris-
tallisiert:

- Cloud-Applikationen operieren meist global. Dies heißt nicht nur, dass der Dienst
    stets über das Internet erreichbar ist. Bei der Betrachtung von globalem Zugriff
    muss berücksichtigt werden, dass der Dienst in lokalen Rechenzentren dupliziert
    werden muss [22]. Nur so kann eine geringe Latenz garantiert werden. Dabei führt
    die Integrität von Daten häufig zu Problem. Die persönlichen Daten müssen dem
    Nutzer stets zur Verfügung stehen. Dem Nutzer darf nicht ersichtlich sein, dass
    seine Daten an mehreren Orten abgespeichert sind.
- Cloud-Applikationen müssen mit vielen tausend von parallelen Nutzern operie-
    ren können. Neben einer vertikalen Skalierung (mehr und bessere Hardware) wird
    demnach auch eine horizontale Skalierung nötig [22]. Diese wird meist durch Paral-
    lelisierung umgesetzt. Eine horizontale Skalierung führt erneut zu einem Fokus auf
    die Konsistenz und Synchronisation des Systems.
- Die Architektur einer Cloud-Applikationen muss mit der Annahme entwickelt wer-
    den, dass die Hardware nicht konstant ist und Fehler vorkommen werden [22].
    Gannon et Al. [22] sieht Probleme bei Applikationen, welche nicht für die Cloud
    entwickelt wurden. Diese nehmen an, dass die Hardware und das Betriebssystem
    konstant sind. Der kleinste Fehler im Rechenzentrum oder des Netzwerks führt hier
    bereits zu Fehlern.
- Cloud-Applikationen müssen die Anforderungen an Verteilte-Systeme beachten
    [44]. Als Beispiel soll das sogenannte CAP-Theorem erwähnt werden [26]. Ohne
    eine Optimierung für den Ausfall einzelner (Teil-)Systeme ist die AnforderungParti-
    tion Tolerancedes CAP-Theorem nicht erfüllt. Es stellt sich also heraus, dass verteilte
    Systeme eine Reihe von Anforderungen mit sich bringen. Die Architektur muss
    demnach damit umgehen, dass sich Dienste nicht auf der selben Maschine befinden.
    Es wird folglich mit einem Netzwerk von Maschinen gearbeitet. Es wird die Welt
    der Verteilten Systeme betreten. Diese bringt eine Reihe von eigenen Anforderun-
    gen mit sich. Oft werden hier Falsch-Annahmen gemacht. Scholl et Al. [44] nennt
    diese AnnahmenFallacies of Distributed Systems. Dazu gehören Aussagen wie unter
    Anderem “the Network is reliable”, “Latency is zero” oder “There is infinite band-
    with”. All diese Aspekte führen zu eigenen Anforderungen, welche die Architektur
    beachten muss.
- Die meisten Cloud-Applikationen werden für den Dauerbetrieb entwickelt. Ein
    zugriff der Nutzer soll stets möglich sein. Hier ist außerdem der Gedanke aufzugrei-
    fen, dass Firmen Verträge mit dem Kunden haben können. Diese beschränken eine
    mögliche Ausfallzeit auf wenige Stunden im Jahr. Ein Einspielen von Updates darf
    demzufolge nicht zu einer Downtime für den Nutzer führen. Zudem müssen diese


### 2.3 Prinzipien einer Cloud Native Architektur


Updates stets getestet werden. Auch Fehler im System oder Abstürze einzelner Kno-
ten dürfen nicht das gesamte System, oder einzelne Funktionen dessen, lahmlegen.
Ein umfangreiche Überwachung des Systems und das vorhalten von Ersatzkonten
sind nur einige der wichtigen Operationen.[22]
Zusammenfassend lässt sich sagen, dass Entwickler und Software-Architekten drei
großen Herausforderungen der Cloud gegenüber stehen. Zunächst müssen sie verteilte
Systeme verstehen. Eine Anforderungen, welche so vorher schon relevant war, allerdings
nun noch prominenter in den Vordergrund dritt. Ferner müssen sich Entwickler mit
neuen Technologien wie Containern vertraut machen. Und abschließend müssen sie sich
bewusst werden, welche Entwurfsmuster in der Cloud funktionieren.

### 2.3 Prinzipien einer Cloud Native Architektur

Nachdem nun die Begriffe Cloud-Computing und Cloud-Applikationen beleuchtet wur-
de, kann eine Architektur für die Cloud betrachtet werden. Das Ziel dieser ist, die Möglich-
keiten der Cloud maximal zu Nutzen. Traditionelle Infrastrukturen (on-premise Service
Modelle) fokussieren sich auf eine feste Anzahl an Hardware-Komponenten und deren
Leistung. Der Cloud-Service-Provider teilt die Leistung allerdings flexibel zu. Folglich
nutzen Cloud-Native-Architekturen Prinzipien, wie das horizontale Skalieren und auto-
matische Ersetzen von abgestürzten Komponenten um Leistung und Elastizität bereit zu
stellen.
Sowohl Scholl et Al.[44] , als auch Microsoft, Amazon und Google [49] definieren eine
Vielzahl von Eigenschaften einer Cloud-Native-Architektur. Im Folgenden werden diese
Eigenschaften beleuchtet.

#### 2.3.1 Automatisierung

Eines der Ziele einer Cloud-Native-Architektur ist es, so viele Aspekte des Systems wie
möglich zu automatisieren. Scholl et Al. [44] bezeichnet diese Eigenschaft als ”Operational
Excellence”. Ganz natürlich kommt hier dasContinuous Integration / Continuous Delivery
(CI/CD)ins Spiel. CI/CD beschreibt das automatisierte bauen, testen und deployen des
Systems. Auch das Zurücksetzen von Updates (Rollback) sollte automatisiert werden.
Neben diesen Aspekten ist auch das Logging zentral.
Eine Überwachung der Anwendung sollte ebenfalls Implementiert werden. Diese führt
dazu, dass Fehler frühzeitig erkannt werden können. Darüber hinaus hat diese auch wei-
tere Vorteile.
Aspekte der Cloud wie das on-demmand-self-service ermöglichen es, im Code der An-
wendung selbst, die Umgebung der Anwendung anzupassen. So kann ein System sich
selbst neue Ressourcen buchen und anwenden. Ein Beispiel wäre hier die Speicherplatz-
überwachung. Ist eine Speichereinheit ausgelastet, müsste ein on-premise Deployment
Modell auf physische Hardware-Erweiterung warten. Durch das Cloud-Computing kann
nun die Software selbst neue Ressourcen buchen. Dies ist führt zu weniger Down-Time
und reduziert die Aufgabe des Personals. Grund-Voraussetzung dafür ist eine planvolle
Automatisierung und Überwachung.
Hinzu kommt außerdem, dass eine Cloud-Native-Applikation ein verteiltes System ist.
Dadurch ergeben sich neue Fehler-Quellen. Diese sind oft schwer zu testen. Eine Cloud-
Native-Architektur sollte auch auf diese automatisiert reagieren.
Weiterhin lässt sich das Skalieren des Systems automatisieren. Durch das herauf Skalieren
bleibt das System weiter erreichbar, und durch das herunter Skalieren werden die Kosten
reduziert.


2 Grundlagen

Scholl et Al. [44] nennt zudem die Anforderung "Document everything". Eine Automa-
tisierung und Überwachung können nur dann sinnvoll und funktional sein, wenn diese
auch klar dokumentiert sind. Jedes Team-Mitglied muss bewusst sein, wie die Umge-
bung der Anwendung definiert ist und wie die Anwendung auf Ereignisse automatisiert
reagiert. Ebenso wie die Anwendung selbst, lässt sich auch das Dokumentieren, mit ge-
eigneten Tools, automatisieren.

#### 2.3.2 Sicherheit

Typisch für Cloud Native Architekturen ist derdefense-in-depthSicherheits-Ansatz [44].
Hierbei ist nicht eine zentrale Instanz für die Sicherheit des Systems zuständig. Jede Kom-
munikation zwischen einzelnen Komponenten des Systems ist abgesichert. Dies schließt
eine sichere Verbindung zu dem Source-Code-Repository mit ein. Jede einzelne Kompo-
nente, auch wenn diese intern ist, sollte kein Grund-Vertrauen zu anderen Komponenten
des Systems haben. Jede Komponente sollte sich vor den anderen Schützen.

#### 2.3.3 Sinnvoller Einsatz von State

Die Speicherung vonStatebeschreibt zum einen die Speicherung von Nutzerdaten (z.B.
die Einträge in dem Warenkorb des Nutzers) und den Status des Systems (z.B. welche
Version des Source-Codes wird gerade im Produktiv-System genutzt). Google beschreibt
den State als den schwersten Aspekt von verteilten Systemen [49]. Daher gilt es nur je-
ne Komponenten des Systems mit State zu füllen, welche diesen tatsächlich benötigen.
Komponenten ohne State können erstens einfacher skaliert werden. Zweitens ist bei fehl-
geschlagenen Instanzen kein State verloren gegangen. Abschließend ist ein Roll-Back zu
einer früheren Version mitstatelessKomponenten problemloser. Auch hier muss kein
inkompatibler State vermieden werden.

### 2.4 Cloud-Native-Technologien

Die Umsetzung einer Cloud-Native-Architektur kann mitCloud-Native-Technologiengelöst
werden.
DieCloud Native Computing Foundation(CNCF) wurde 2015 durch die Linux Foundation
gegründet [7]. Weitere Gründungsmitglieder waren unter anderem Google, RedHat, Hua-
wei und Twitter. Neben Beispiel-Projekten, einer Liste von Cloud-Native-Technologien
und weiteren Ressourcen definiert die CNCF auch Cloud-Native-Technologien.
Diese ”[...] ermöglichen es Unternehmen, skalierbare Anwendungen in modernen, dyna-
mischen Umgebungen zu implementieren und zu betreiben.”[6]
Als moderne Umgebungen sind Deployment-Modelle wie die private Cloud, public
Cloud und hybrid Cloud zu verstehen. Als Best-Practices sieht die CNCF unter ande-
remMicro-Services,ContainerundServices-Meshes, auf welche im Folgenden eingegangen
werden soll. Das nachfolgende Kapitel orientiert sich inhaltlich am Buch ”Cloud Native”
vonScholl, Swanson und Jausovec[44].

#### 2.4.1 Micro-Services-Architektur

Eine fundamentale Eigenschaft der Cloud-Native-Architektur ist dieVerteiltheitihrer Be-
standteile. Ein sogenanntes "verteiltes System"beschreibt eine Zusammenkunft aus meh-
reren Untersystemen oder Rechnereinheiten, welche nach außen hin wie eine abgeschlos-
sene Einheit funktioniert. Seine Bestandteile sowie deren Aufbau und Funktion sind dem



2.4 Cloud-Native-Technologien

Nutzer des Systems im Idealfall also nicht bewusst. Die Cloud-Native-Architektur ver-
wendet zur Umsetzung der gesamten Funktionalität des zu schaffenden Systems – und
damit seiner Untersysteme – sogenannte ”Micro-Services”.
Ein Micro-Service ist ein isolierter Kleinstbaustein, welcher im Idealfall genau eine Funk-
tionalität realisiert. Seine Entwicklung, Wartung und Dokumentation werden dabei typi-
scherweise von genau einem Team übernommen. Da jeder einzelne Micro-Service einfa-
cher zu verstehen ist, als das gesamte System, ermöglicht die Micro-Service Architektur
also eine Expernbildung in den einzelnen Teams und verhindert somit eine redundante
Wissensüberschneidung in den Beauftragten des Systems. Die Teams können ihren Fokus
also vollständig auf den ihnen zugeteilten Micro-Service legen.
Zur Kommunikation zwischen einzelnen Micro-Services stellt ein solcher eine klar defi-
nierte Schnittstelle bereit, über die Daten ausgetauscht werden können. [5]


Entwickler verschiedener Micro-Services benutzen die Schnittstellen der jeweils Anderen,
um Datenflüsse abzubilden, die zur Umsetzung des eigenen Micro-Service vonnöten sind.
Gleichzeitig stellen sie für andere Micro-Services dem von ihnen entwickelten Micro-
Service ebenfalls eine Schnittstelle bereit.
In der Theorie ist für einen ServiceAist nicht von Bedeutung, wie ein anderer ServiceB
seine Funktionalität realisiert. Seine Schnittstelle und deren Definition istAbekannt.A
benutzt die Schnittstelle vonBbeispielsweise, um Daten abzufragen, die zur Berechnung
einer hypothetischen FunktionFAbenötigt werden. Micro-ServiceAstelltFAwiederum
über eine Schnittstelle nach außen hin zur Verfügung. Die Nutzung einer Schnittstellen-
funktion eines Micro-Service erfolgt entweder programmatisch durch den Zugriff eines
anderen Micro-Service, oder über Anknüpfung an eine grafische Benutzeroberfläche.


Diese Kapselung ermöglicht eine einwandfreie, getrennte Weiterentwicklung spezifischer
Micro-Services, sofern sich die Schnittstellendefinition verschiedener Versionen nicht un-
terscheidet. Eine vollständige Äquivalenz zwischen den Schnittstellen verschiedener
Micro-Service-Versionen kann jedoch nicht immer gewährleistet werden.


2 Grundlagen

Zur Lösung dieser Problematik schlägtJean-Jacques Dubrayverschiedene Lösungsansätze
vor [17][44]. Eine Möglichkeit zum Design von Schnittstellen und deren Versionen ist
das sogenannte ”Knoten-Modell”. In ihm sind benutzende Micro-ServicesB 1 , ...,Bneiner
SchnittstelleSfest an ihre Version gebunden. Sie besitzen keinerlei Garantie über die
Äquivalenz der Schnittstelle in kommenden Versionen. Ändert sich die Schnittstelle, müs-
sen alle von ihr abhängigen Benutzer ebenfalls manuell überprüft und geändert werden.
Eine weitere Strategie ist die sogenanntePunkt-zu-Punkt-Methode. Ähnlich zum Knoten-
Modell sind alle Benutzer einer bestimmten Schnittstelle zunächst an eine bestimmte
Version gebunden. Im Gegensatz zum Knoten-Modell bleiben Micro-Services mit veral-
teten Schnittstellenversionen bei deren Änderung jedoch bestehen. Die Teilnehmer eines
vorhandenen Netzes von miteinander verbundenen Micro-Services müssen also nicht so-
fort auf eine neue Version eingestellt werden. Einzelne Netzteilnehmer können bei Bedarf
auf die neue Schnittstellenversion ihrer Partner aktualisiert werden. Die Micro-Services
unterschiedlicher Schnittstellenversionen werden dabei parallel betrieben, damit der Zu-
griff auf veraltete Versionen dauerhaft gewährleistet werden kann.
Die letzte von Dubray vorgeschlagene Lösungstrategie beschreibt das Modell einer ”kom-
patiblen Versionierung”. Anstelle einer festen Verbindung unterschiedlicher Benutzer
an die Schnittstelle einer bestimmten Version sind letztere hier an alle Endpunkte der
Schnittstellendefinition gebunden, die zu einem bestimmten Versionszeitpunkt verfügbar
waren. Alle Folgeversionen müssen die Idempotenz der Funktionalität dieser Endpunkte
hinsichtlich der vorangegangenen Versionen gewährleisten. Um einer Schnittstelle neue
Funktionalität hinzuzufügen stellt man deswegen gänzlich neue Endpunkte zur Verfü-
gung. Man spricht konkret von der ”Abwärtskompatibilität” neuerer Schnittstellenversio-
nen.

**Kommunikation unter Micro-Services**

Zum Datenaustausch bedienen sich Micro-Services verschiedener Netzwerkkommunika-
tionsmodelle. Dabei unterscheidet man zwischeninterneroderhorizontalerundexterner
beziehungsweisevertikalerKommunikation. Während erstere die Kommunikation zwi-
schen Micro-Services einer Gruppierung von Micro-Services beschreibt, stellt letztere den
Informationsaustausch zwischen einer solchen Gruppierung und externen Dienstleistern
oder Nutzern dar. Eine Möglichzeit zur Umsetzung dieses Modells findet sich in Kapitel
4.2.1.
Zum konkreten Datenaustausch bedienen sich Micro-Services dem ”Client-Servier” Mo-
dell. Den Kommunikationspartnern steht dabei unter anderem das ”Request-Response”-
Modell zur Verfügung. Ein MicroserviceA, der Daten oder Funktionen bereitstellt, setzt
diesen Vorgang über einen Server um. Dieser Server besitzt eine feste Adresse. Er stellt
spezifische Datensätze oder Funktionen über einen oder mehrere, festgelegte Endpunkte
bereit. Ein Client ist in der Lage, durch die Kombination einer Serveradresse und eines
Endpunkt Anfragen nach diesen Daten oder der Ausführung einer Funktion zu stel-
len(Request). Aufgabe des Servers ist es danach, eine sinnvolle Antwort an den Client
zurückzusenden (Response). Um den Abrufungsprozess weiterhin zu steuern, besitzt ein
Client die Möglichkeit, seinen Anfragen bestimmte Parameter hinzuzufügen.As Schnitt-
stelleSAsetzt sich im ”Response-Request-Modell” aus allen Endpunkten zusammen, die
Aüber seinen Server nach außen hin verfügbar macht.SAlegt weiterhin fest, ob und
welche Parameter sowie deren Aufbau ein in ihr enthaltener Endpunkt besitzt.
Eine weitere Möglichkeit zum Datenaustausch bietet das ”Publisher-Subscriber-Modell”.
In diesem benutzt der Micro-ServiceP, der Daten bereitstellt, einen sogenannten ”Message-
Broker”. Dieser Message-Broker stellt eine Art Kanal (Topic)zur Verfügung, in welchenP



2.4 Cloud-Native-Technologien

Daten oder Nachrichten zur Verfügung stellt. Diesen Vorgang nennt man ”Publishing”.
Eine beliebige Anzahl an weiteren Micro-ServicesS 0 , ...,Sn ist in der Lage, sich beim
Message-Broker für diese Topic zu registrieren. WennPeine Nachricht in diesen Kanal
hinterlegt, werden alle registrierten ServicesS 0 , ...,Snbenachrichtigt. Diese sind dann in
der Lage, die so gewonnenen Daten weiterzuverarbeiten.

#### 2.4.2 Container

EinContainerist eine Einheit, welche Software, ihre Abhängigkeiten sowie Konfigura-
tionen in sich bündelt. Er wird wiederum innerhalb einesContainer-Abbildesgebündelt.
Ein Container-Abbild ist eine Art Vorlage, die den Erstellungsprozess eines Containers
beschreibt. Es enthält weiterhin zusätzliche Informationen, die zur Ausführung der im
Container enthaltenen Software benötigt werden. Unter diesen befinden sich unter ande-
rem verschiedene Systemtools und -bibleotheken, sowie eine Laufzeitumgebung. Wäh-
rend Micro-Services die Funktionen einer Cloud-Native-Architektur einzeln realisieren,
kapseln Container einzelne Micro-Services wiederum vom Betriebssystem ihres ausfüh-
renden Computers ab. Da Container-Abbilder alle nötigen Informationen zur Ausführung
von Software beinhalten, können in ihnen gekapselte Micro-Services betriebssystemun-
abhängig ausgeführt werden, solange auf dem ausführenden System eine sogenannte
Engineexistiert, welche eine Brücke zwischen seinem Betriebssystem und dem Container
liefert. Container sind durch diesen Vorgang vom ausführenden System und anderen
Containern isoliert. Möchte Software innerhalb eines Container in ihm beinhaltete Daten
oder Abhängigkeiten verändern, geschieht das durch die sogenannte ”Copy-on-Write”-
Strategy (COW). Die zu verändernden Daten werden zunächst kopiert, bevor sie ver-
ändert werden. Je nach Implementierung liefern verschiedene Container-Architekturen
unterschiedliche Strategien, um veränderte Daten nachhaltig zu persistieren. In der Micro-
Service-Architektur werden diese Stragien in der Regel selten benötigt, da ihre Bestand-
teile meist zustandslos sind. Durch die in diesem Abschnitt vorgestellten Methoden sind
Container in der Lage, ihre Funktion unabhängig der zugrunde liegenden Infrastruktur
auszuführen. [14].

#### 2.4.3 Service-Meshes


Um die Vorteile und Eigenschaften eines Service-Meshes zu betrachten, muss zunächst
der Begriff Container-Orchestration beleuchtet werden.

2.4.3.1 Container-Orchestration

Werden die Services einer Micro-Service-Architektur in Containern betrieben, können sie
unabhängig von einer festen Infrastruktur betrieben werden.
Für eine Cloud-Native-Architektur ist dies von Vorteil. Denn Container lassen sich mit
minimaler Verzögerung starten und stoppen. Zudem können wir horizontal skalieren,
indem wir weitere Container starten. Allerdings ergeben sich dadurch einige Probleme.
Zunächst ein Mal müssten Container automatisch gestartet, gestoppt und neu gestartet
werden. Die Skalierung des Systems, oder im Fehlerfall das Neu starten von Containern
darf keine menschliches Aufgabe sein. Denn dies wäre zwangsläufig mit einer zu großen
Verzögerung verbunden.
Um diese Verzögerung weiter zu reduzieren, könnten bereits Ersatz-Container vorge-
halten werden. Im Fehlerfall oder für die Skalierung würde ein Umleiten des Traffics
genügen. Dies erfordert ein automatisiertes Load-Balancing.



Ein funktionierendes Load-Balancing ist auch für die Container zu Container Kommu-
nikation notwendig. Wird ein Service durch mehrere Container umgesetzt, gilt es zu
definieren, welcher dieser Container adressiert werden soll. Dieses Problem wird als
internal Load-Balancing bezeichnet.

DieContainer Orchestration[44] behebt diese Probleme. DerContainer-Orchestratorliegt als
eine Schicht über den Containern. Er ist verantwortlich für das starten und stoppen von
Containern. Außerdem teilt er flexibel Rechenleistung zu und skaliert das System passend
zum bestehenden Traffic.
Stehen mehrere Server zur Verfügung, platziert er neue Container auf den Servern mit
der niedrigsten Auslastung. Außerdem verschiebt er Container, sollte die Auslastung zu-
nehmen. Auch andere Metriken sind für die Server Auswahl und die Verschiebung der
Container möglich.
Der Container-Orchestrator überwacht zudem die Gesundheit der Container. Falls nötig
startet er diese neu.
Abschließend ist er für das interne und externe Load-Balancing verantwortlich. Realisie-
ren mehrere Container den selben Service, leitet der Load-Balancer den Traffic zu dem
geeignetsten Container weiter.
Neben den genannten Funktionen kann der Container-Orchestrator auch für das State-
Management und viele weitere Aspekte des Systems zuständig sein.
Scholl et Al.[44] sieht in Google Kubernetes [28] die populärste Container Orchestrator
Wahl.

**2.4.3.2 Herausforderungen der Micro-Service-Architektur**

Durch die Aufteilung unserer Anwendung in Micro-Services haben wir die Komplexität
der einzelnen Bestandteile unserer Architektur reduziert. Man sollte allerdings bedenken,
dass gleichzeitig die Komplexität auf dem System-Level zugenommen hat. Jeder Service
einer Micro-Service Architektur wird, neben der Business Logik, unter anderem folgendes
enthalten [8]:

- Jede Service wird eine Retry-Logik umsetzen. Es ist möglich, dass Services nicht
    erreichbar sind. Ist dies der Fall, muss definiert werden, wie oft eine Verbindungs-
    versuch unternommen wird. Ferner ist eine Ausnahmebehandlung zu ergänzen.
- Darüber hinaus ist die Umsetzung der Sicherheit zu beachten. Eine Firewall kann
    die Anwendung von außen Schützen. Allerdings gilt es das Konzept der in-depth
    Security umzusetzen. Diese verlangt, dass auch die Kommunikation unter den Con-
    tainern abgesichert ist. Ferner sollte nicht jeder Container frei mit jedem anderen
    Container kommunizieren dürfen.
- Abschließend ist für die Automatisierung einer Überwachung unumgänglich. Es
    gilt Metriken zu erfassen und auf Basis dessen zu Handeln. Logger und Logik zum
    Fehlerfinden müssen ebenfalls in jedem Service integriert werden.

Ein Vorteil der Microservice-Architektur war es, dass jedes Team einen klar getrennten
Anteil der Anwendung entwickelt. Allerdings muss jedes Team, neben der eigentlichen
Business Logik, die zuvor genannten Funktionen entwickeln, verwalten und konfigurie-
ren. Diese Konfiguration kann nicht individuell für jeden Service statt finden, ohne zu
Fehlern und Problem zu führen. Außerdem wird die Fehlersuche deutlich erschwert. Eine
Isolation von Fehlern in einem einzigen Service ist nicht mehr sichergestellt.



2.4 Cloud-Native-Technologien

**2.4.3.3 Vorgehensweise von Service-Meshes**
Service Meshesnutzen einen sogenanntenSidecar-Proxy[8]. Dieser wird in Abbildung
2.2 dargestellt. Jedem Container wird ein weiterer Container hinzugefügt. Dieser fun-
giert als Proxy. Er nimmt demnach Verbindungen an und leitet sie zu dem eigentlichen
Service weiter. Diese Side-Car-Proxys werden durch dieControl-Planeverwaltet und in
die einzelnen Services injiziert. Ein neuer Service muss demnach die zuvor genannten
Probleme, wie Sicherheit oder die Retry-Logic nicht erneut umsetzen. Außerdem ist die
Control-Plane eine zentrale Schnittstelle um die gesamte Anwendung zu konfigurieren
und zu Überwachen. Neben den genannten Funktionen, kann ein Service-Mesh auch
weitere Aspekte, wie zum Beispiel das Load-Balancing, lösen. Meist wird an dieser stelle
aber auf einem Container-Orchestrator wie Kubernetes [28] aufgebaut. Eine mögliches
Service-Mesh wäre zum Beispiel Istio [25].


## 3 Analyse

### 3.1 Differenzierung

Dieses Kapitel vergleicht die Prinzipien, Eigenschaften und Umsetzungskriterien der
Cloud-Native-Architektur mit denen eines Software-Monolithen und stellt deren Un-
terschiede dar. Er orientiert sich inhaltlich an dem den BlogpostsPattern: Monolithic
Architecture[42] undPattern: Microservice Architekture[5] vonmicroservices.io, einem Ver-
gleich von Microservices und Monolithen der WebseiteMuleSoft[36].

Ein grundlegender Unterschied zwischen einer monolithischen Softwarearchitektur und
der einer Cloud-Native-Architektur findet sich in der Zusammensetzung ihrer Komponen-
ten. Ein sogenannterMonolithist ein einheitliches Softwaresystem, welches die gesamte
Programmlogik, Datenverwaltung und die Benutzeroberfläche in einer abgeschlossenen,
ausführbaren Datei verbindet. Die in ihr enthaltenen Komponenten sind oft eng mitein-
ander verknüpft. Diese Zusammensetzung bietet einen in den Anfangsphasen der Ent-
wicklung simplen Implemtierungsprozess, da die Entwickler während der Umsetzung
eines Prototyps unmittelbar Änderungen an den verschiedenen Komponenten umsetzen
können. In den Anfängen eines Projektes ist es zudem simpel, Änderung zu veröffent-
lichen. Dazu muss lediglich eine neue Version der gebündelten Datei des Monoliths an
seine Nutzer verteilt werden.
Ein zunehmendes Wachstum des Monoliths fördert jedoch eventuell einige, nicht uner-
hebliche Nachteile. Erhöht sich die Größe des Monoliths und seiner Codebasis, wächst
die Wahrscheinlichkeit von abnehmender Modularität seiner Bestandteile. Wenn sie stark
miteinander verwoben sind, steigt die Komplexität bei notwendigen Veränderungen
oder der Einführung neuer Funktionen an. Gleichzeitig wächst die Einarbeitungszeit
in das Projekt für neue Teammitglieder. Es besteht weiterhin die Möglichkeit, dass sich
der Softwaremonolith in einen ”Big Ball of Mud” verwandelt. ”Ein Big Ball of Mud
ist ein planlos strukturierter, ausufernder, schlampiger, mit Klebeband und Bindedraht
zusammengehaltener Spaghetti-Code-Dschungel.”[21] Mit zunehmender Seniorität der
Mitglieder des Entwicklungsteams besteht außerdem die Gefahr, dass nur wenige oder
gar keine Entwickler die gesamte Funktion des Monolithen vollständig nachvollziehen
können. Schlussendlich ist man in der Entwicklung eines Monoliths meist unabdinglich
an seine ursprünglich ausgewählte Technologieplatform gebunden. Der dichte Verbund
der Komponenten des Monoliths erschwert den Austausch einzelner Technologien erheb-
lich.


Der Aufbau eines Cloud-Native-Softwareprojektes steht im direkten Gegensatz zur mo-
nolitischen Softwarearchitektur. Es besteht aus individuellen, lose gekoppelten Kleinst-
bausteinen. Ihre Berührungspunkte untereinander bestehen einzig und allein aus ihren
nach außen hin verfügbaren Schnittstellen. Aufgrund dieser Tatsache ist die Modurarität
und Abgeschlossenheit der Bestandteile dauerhaft gewährleistet. Weiterhin ermöglicht
die Kapselung der verschiedenen Micro-Services eine programmiersprachenagnostische
Entwicklung und somit die Auswahl des ”Best Tool For The Job”.


3 Analyse

Die Komplexität der Bestandteile eines Cloud-Native-Softwaresystems besteht im Gegen-
satz zu denen eines Monolithen nur aus der Komplexität der einzelnen Micro-Services.
Da die Schnittstellen der verschiedenen Micro-Services fest definiert sind, kann bei der
Betrachtung eines einzelnen Micro-Services die interne Funktionsweise seiner Interak-
tionspartner gänzlich außer Acht gelassen werden. Im Idealfall wird ein Micro-Service
außerdem von genau einer festen Gruppe oder eines festen Teams von Entwickler ver-
waltet. Deren Mitglieder benötigen dann nur das Wissen über die Funktionalität des
von ihnen betreuten Bestandteils der gesamten Architektur. Diese Tatsachen erleichtern
den Entwicklungsprozess bei Fehlerbehebungen, der Erweiterung eines Micro-Service
um zusätzliche Funktionen und das Testen der unterschiedlichen Micro-Services als
abgeschlossene Einheit. Im konkreten Testprozess eines einzelnen Micro-Service kann die
Funktion anderer, zum Funktionsaufruf benötigter Micro-Services durch den Program-
mierer simuliert werden. Diese Aufteilung hat jedoch den Nachteil, dass die Komplexität
des Gesamtsystems direkt im Zusammenhang zur Anzahl und Qualität der Dokumenta-
tion seiner Einzelteile steht. Je größer die Anzahl unterschliedlicher Micro-Services, desto
schwieriger ist es für die Beauftragten des Softwaresystemes, einen Überblick zu behalten.
Bei sehr großen Systemen steigt die Gefahr von Quelltextduplizierung und vermindeter
Auffindbarkeit bereits existierender Micro-Services, die eine benötigte Funktionsweise
bereits abdecken.
Solange sich die Schnittstellen der Micro-Services nicht ändern, kann die Weiterent-
wicklung unabhängig von anderen Implementierungsteams geschehen. Strategien zur
Anpassung des Gesamtsystems nach der Veränderung einer Schnittstelle finden sich in
Kapitel 2.4.1. Diese im vorangegangenen Absatz genannten Punkte lassen sich unter den
BegriffenAgilitätundContinous Innovationzusammenfassen.
Im Kontrast zur monolitischen Softwarearchitektur ermöglicht der Aufbau der Micro-
Service-Architektur und die Bereitstellung von Micro-Services auf Cloud-Hostern einen
dynamischen Austausch ihrer Bestandteile zum Zeitpunkt der Veröffentlichung neuer
Versionen. Damit muss ein Benutzer des Softwaresystems bei Änderung seiner Kompo-
nenten nur selten in Kenntnis gesetzt werden. Dieser Prozess ermöglicht die Minimierung
von Fragmentierungen zwischen den unterschiedlichen, aktiv genutzten Versionen eines
Softwaresystems, da es weitestgehend automatisiert aktualisiert werden kann. Dieser
Prozess wird mit dem BegriffContinous Deliverybezeichnet.

Die Cloud-Native-Architektur unterscheidet sich von einer monolitischen weiterhin
durch den Prozess der Skalierung. Der traditionelle Skalierungsansatz eines Monoliti-
hen ist die parallele Ausführung mehrerer Instanzen und die subsequente Verteilung
von Anfragen über einen ”Load-Balancer”. Da der Monolith jedoch ein festes Bündel
seiner Bestandteile ist, ist es nur möglich, ihn als Ganzes zu skalieren. Die Aufteilung
der Micro-Services der Cloud-Native-Architektur erlaubt es hingegen, jeder Komponente
genau nur die benötigten Ressourcen zuzuweisen. Dieses Vorgehen erlaubt eine dynami-
sche Umverteilung von bereits vorhandener Ressourcen und eine Zuweisung gänzlich
neuer Rechen- und Speicherleistung bei veränderter Lastverteilung oder einer Welle von
neuen Anfragen. Im Ideallfall werden solche Skalierungsprozesse vom verwendeten
Cloud-Hoster vollautomatisiert umgesetzt. Ihre dynamische Natur erlaubt weiterhin
flexible Bezahlungsmodelle. Im sogenannten ”Pay-As-You-Go”-Modell bezahlt ein Nut-
zer eines Cloud-Hosters nur genau die Ressourcen, die zur Ausführung seiner Software
genutzt werden. Die Zustandslosigkeit der Bestandteile einer Cloud-Native-Architektur
erlaubt eine Fokussierung auf die verwendetenDatastoresbei der Umsetzung effizienter
Caching-Strategien.


### 3.2 Zusammenfassung

### 3.2 Zusammenfassung

Zusammenfassend kann man sagen, dass sowohl Cloud-Native- sowie monolitische Soft-
warearchitekturen einige Vor- und Nachteile besitzen, die vor dem Start eines Projektes
sorgfältig gegeneinander abgewogen werden müssen. Während ein Monolith bei Projekt-
start oder in kleineren Teams oft eine schnelle Entwicklung eines Prototypen ermöglicht,
erleichtert ein gut geplanter Aufbau von größeren Softwareprojekten mit Cloud-Native-
Technologien eventuell unter anderem die Verteilung verschiedener Mitarbeiter innerhalb
des Projektes oder die Auslieferung der Software an seine Nutzer.



## 4 Tools und Industriestandards


In diesem Kapitel werden die Tools und Industriestandards betrachtet, welche für das
Umsetzen der in Kapitel 2.4 vorgestellten Cloud-Native Technologien zum Einsatz kom-
men. In der Praxis gibt es viele verschiedene Projekte und Ansätze, Container sowie
Container-Orchestratoren zu realisieren, die jeweils eigene Vor- und Nachteile mit sich
bringen. Zusätzlich existiert heutzutage eine Vielzahl an Cloud-Anbietern, welche ihre
Hardwareressourcen für das Aufsetzen von Cloud-Native-Anwendungen zur Verfügung
stellen und dazu diverse Dienste für das einfachere Betreiben dieser anbieten.

Im Folgenden werden die am häufigsten verwendeten Tools und Industriestandards
vorgestellt und anschließend ein Überblick darüber verschafft, in welchen Industrien und
Branchen Cloud-Native-Anwendungen eingesetzt werden.

### 4.1 Container Runtime Environments

Von Vielen wird der Begriff „Container“ schnell mit Docker assoziiert. Hierbei handelt
es sich jedoch bei weitem nicht um das einzige Projekt, welches das Konzept der Con-
tainervirtualisierung realisiert. Neben dieser wohl bekanntesten Container Runtime von
Docker Inc. gibt es eine Vielzahl an weiteren Tools, welche ebenfalls die Möglichkeit
bieten, Applikationen in Containern zu isolieren.


Bevor wir die verschiedenen Container Runtimes genauer betrachten, muss zunächst
definiert werden, was man unter einer Container Runtime versteht.

#### 4.1.1 Container Runtime

Eine Container Runtime hat Ähnlichkeiten zu der Runtime einer gewöhnlichen Software-
Applikation. Es handelt sich hierbei um Software, welche die einzelnen Komponenten,
die für das Starten eines Containers benötigt werden, verwaltet und dafür sorgt, dass
diese funktionsfähig sind [19]. Sie ermöglicht, dass der Nutzer sich nicht selbst um das
Management dieser Komponenten kümmern muss, wodurch das Arbeiten mit Containern
und deren Deployment effizienter durchgeführt werden können.

#### 4.1.2 Linux Control Groups (cgroups)

Im Jahr 2007 wurden in den Linux Kernel sogenannte cgroups (Linux Control Groups)
integriert [19]. Hierbei handelt es sich um ein Linux Kernel Feature, welches ermöglicht,
Prozesse in hierarchischen Gruppen zu organisieren, deren Nutzung von verschiedenen
Ressourcen limitiert und überwacht werden kann [4].
Auf Basis dieses neuen Features wurden anschließend einige erste Projekte ins Leben ge-
rufen, welche containerisierte Prozesse ausführen konnten. Hierzu zählen unter anderem
LXC, LMCTFY, systemd-nspawn oder rkt [19].


4 Tools und Industriestandards

#### 4.1.3 Linux Containers (LXC / LXD)

LXC, kurz für Linux Containers, wurde kurz nach cgroups vorgestellt und befindet sich
seit 2008 in Entwicklung [30]. Das Projekt wurde hauptsächlich in C, Shell und Python
geschrieben, und verwendet ein eigenes Containerformat namens Linux-Containers. Der
Fokus von LXC liegt auf sogenannten Full-System-Containers [30]. Hierbei handelt es
sich um Container, die darauf abzielen, ein gesamtes Betriebssystem zu virtualisieren.
Diese Art von Containern ist für Cloud-Anwendungen eher ungeeignet, da hier kein
ganzes Betriebssystem virtualisiert werden muss, sondern lediglich ein einzelner Prozess
abgekapselt ausgeführt werden soll.

LXC hat später mit LXD (Lexdi) noch ein Folgeprojekt erhalten, welches ebenfalls auf
Full-System-Container spezialisiert ist. Letztendlich hat sich LXC aber, ebenso wie der
Ansatz von systemd, systemd-nspawn, nie bei den Endnutzern durchgesetzt. Sie wurden
jedoch von anderen Systemen verwendet. Docker wurde beispielsweise teilweise auf
Basis von LXC gebaut [19].

#### 4.1.4 LMCTFY (Let Me Contain That For You)

LMCTFY (Let Me Contain That For You) war ein frühes Container-Projekt von Google,
welches zum Großteil in C++ geschrieben ist. Der Fokus lag hierbei darauf, ähnlich wie
bei Docker, einzelne Applikationen innerhalb eines Containers isoliert auszuführen. Das
Projekt ist Open Source, wurde jedoch eingestellt als Docker zunehmend an Popularität
gewann [29].

#### 4.1.5 Docker

Docker ist wohl das bekannteste Projekt für Containervirtualisierung. Es wurde zu Beginn
unter dem Namen dotCloud entwickelt und verwendete zunächst LXC als Basis. Nach
kurzer Zeit gründete Docker zusammen mit einigen anderen Unternehmen die Open
Container Initiative (OCI), um einige Standards für Containertechnologien zu definieren,
worauf hin LXC durch eine eigens entwickelte Schnittstelle namens libcontainer ersetzt
wurde [19].

Docker wird heutzutage von vielen Nutzern mit dem Begriff Containervirtualisierung
gleichgesetzt und bietet neben einer Container Runtime viele weitere Features. Die hausei-
gene Container Runtime unterstützt Container in einem eigens von Docker entwickelten
Format (Docker-Container). Das Tool ist in Go geschrieben und wird von Docker Inc.
entwickelt. Docker unterstützt keine Full-System-Container und fokussiert sich auf die
Isolierung einzelner Applikationen, wodurch es für die Verwendung bei Microservices
und Cloud-Native-Architekturen geeignet ist.

Das Docker-Tool beinhaltet neben dem Starten eines Containers viele weitere Featu-
res, wie beispielsweise das automatische Erstellen von Images, auf dessen Basis dann ein
Container erstellt werden kann, oder das Hochladen eines solchen Images in das soge-
nannte Docker Hub. Hierbei handelt es sich um eine große Sammlung von Docker-Images,
welche gratis verwendet werden können. Fast alle großen IT-Projekte, wie beispielsweise
bekannte Datenbanken wie MySQL oder Redis, sind hier mit einem Docker-Image vertre-
ten [15]. Mit Hilfe des Docker-Tools kann man durch ein einfaches Ausführen von „docker
run“ auf das jeweilige Image sofort einen Container auf dessen Basis erstellen, wobei
Docker alle benötigten Abhängigkeiten automatisch herunterlädt. Somit können all diese


### 4.1 Container Runtime Environments

Tools ohne eine komplizierte Installation sofort verwendet werden. Als Nutzer hat man
zusätzlich die Möglichkeit, seine eigenen Images gratis auf Docker Hub hochzuladen, in
private oder öffentliche Registries. Auf öffentliche Registries haben dann auch andere Nut-
zer Zugriff und können diese Images verwenden. Dabei bietet Docker Hub zusätzlich eine
Versionierung von Images, indem diese mit einem Versions-Tag versehen werden können.

Des Weiteren bietet Docker mit Docker-Compose ein Tool zum Starten von mehre-
ren Containern gleichzeitig, wobei Abhängigkeiten und Verbindungen zwischen diesen
realisiert und berücksichtigt werden können. Zusätzlich gibt es mit Docker Swarm ein
hauseigenes Orchestrierungs-Tool. Docker ist somit ein leicht zu bedienendes All-In-One
Werkzeug für Containervirtualisierung.


Diese Einfachheit hat stark zu der heutigen Beliebtheit von Docker beigetragen. Außer-
dem wird Docker von vielen Cloud-Anbietern und Container-Orchestratoren unterstützt.

#### 4.1.6 rkt („Rocket“)

Mit rkt („Rocket“) wurde von den Entwicklern der Linux Distribution CoreOS eine
Alternative für Docker entwickelt. Auch hier wurde als Programmiersprache Go verwen-
det [43]. Zur damaligen Zeit enthielt rkt einige Features, welche Docker und anderen
Container Runtimes überlegen waren. Beispielsweise war es bei rkt nicht notwendig,
alle Prozesse als root zu starten [19]. Außerdem unterstützt rkt neben ihrem eigenen
Containerformat auch weitere Formate, unter anderem Docker-Container.


CoreOS hat zunächst mit appc einen eigenen Container-Standard veröffentlicht, hat
sich jedoch später bei der Gründung der Open Container Initiative dieser als Gründungs-
mitglied angeschlossen [19]. Daraufhin war rkt eine Zeit lang der größte Konkurrent
von Docker. Nachdem letzteres jedoch immer mehr an Popularität gewann, wurde rkt
letztendlich 2020 eingestellt und wird aktuell nicht mehr aktiv weiterentwickelt [43].

#### 4.1.7 Podman


Podman steht für Pod Manager und ist eine von Redhat entwickelte Docker Alternative.
Der Fokus liegt hierbei, wie der Name bereits andeutet, auf sogenannten Pods. Hierbei
handelt es sich um eine Abstraktion über einem Container oder mehreren Containern,
die sich gewisse Ressourcen teilen. Der Begriff stammt aus dem Kubernetes-Umfeld.

Podman ist in Go geschrieben [40]. Redhat kritisiert an Docker, dass es mit der Do-
cker Engine ein All-In-One-Tool für Containervirtualisierung bietet. Man möchte mit
Podman die UNIX-Philosophie verfolgen, welche besagt, dass ein Programm immer
eine Aufgabe erledigen soll. Somit wird Podman nur für das Starten und Managen von
Containern bzw. Pods verwendet, nicht aber beispielsweise für das Bauen von Images.
Hierfür gibt es wiederum ein eigenes Tool, welches sich darauf spezialisiert, wie zum
Beispiel buildah. [19].

#### 4.1.8 Open Container Initiative Runtimes

Wie bereits erwähnt haben sich einige Unternehmen zusammengeschlossen und die Open
Container Initiative gegründet. Die hierbei entstandenen Spezifikationen beschreiben


4 Tools und Industriestandards

eine sogenannte Low-Level-Runtime. Dies bedeutet, die Runtimes fokussieren sich auf
das Management des Lebenzyklus von Containern und müssen sich ansonsten um nichts
weiteres kümmern [19]. Sie werden somit meist nicht vom End-User direkt verwendet,
sondern sind in eine andere High-Level-Container-Software, wie beispielsweise Docker,
eingebettet.

Die bekannteste Low-Level-Runtime ist runC. Hierbei handelt es sich um die OCI
Runtime von Docker, welche im Rahmen von libcontainer entstanden ist. Das Projekt ist
in Go geschrieben und Open Source verfügbar [19].

Oracle hatte mit Railcar eine Alternative zu runC ins Leben gerufen, welche ebenfalls
auf die Spezifikationen der OCI aufsetzt. Als Programmiersprache wurde Rust gewählt,
welche sich ebenso wie Go als sehr geeignet für Containervirtualisierung herausstellte.
Trotzdem wurde das Projekt später eingestellt [19].

Auch Redhat hat eine eigene Implementation der OCI Spezifikationen entwickelt. Die
Runtime heißt crun und wurde in C geschrieben. Laut dem offiziellen GitHub Repository
ist crun schneller als runC und benötigt weniger Ressourcen. Man hat sich bewusst für
C als Programmiersprache entschieden, da C leichtgewichtiger als Go ist und die in Go
geschrieben Runtimes meist selbst auf C-Module zugreifen, um ihre Container zu starten
[12].

#### 4.1.9 Kubernetes Container Runtime Interface (CRI)

Kubernetes hat als bekanntester Container Orchestrator mit dem Release von Kuber-
netes 1.5 das Container Runtime Interface eingeführt. Hierbei handelt es sich um ein
Plugin-Interface, welches Kubernetes ermöglicht verschiedene Container-Runtimes zu
verwenden, indem sie dieses Interface implementieren [10]. Das CRI erweitert die von
der OCI definierten Spezifikationen um einige für die Orchestrierung relevanten Funktio-
nalitäten.

Eine Implementation des CRI ist containerd. Es handelt sich dabei um die High-Level-
Runtime von Docker, welche als Low-Level-Runtime standardmäßig runC verwendet
[19]. Das Projekt wurde wie Docker in Go geschrieben und unter einer Open-Source
Lizens losgelöst von Docker auf GitHub veröffentlicht [9].

Mit CRI-O hat auch Redhat eine Implementation des CRI veröffentlicht. CRI-O soll
als Brücke zwischen dem CRI und OCI Runtimes fungieren und ermöglicht das Einbin-
den von allen OCI Runtimes in Kubernetes und ist dabei leichtgewichtiger als Docker
[11].

### 4.2 Container Orchestrator

Auch für die Container Orchestrierung gibt es einige verschiedene Produkte auf dem
Markt. Das bereits erwähnte Kubernetes ist das am häufigsten eingesetzte Tool und hat
sich mittlerweile als Industriestandard durchgesetzt. Es existieren jedoch noch weitere
Projekte, von denen einige in diesem Kapitel vorgestellt werden.


#### 4.2.1 Kubernetes


Kubernetes ist ein Open-Source Container-Orchestrator. Das Projekt wurde von Google
entwickelt und 2014 erstmals veröffentlicht [27]. Wie bei Docker wurde auch hier Go als
Programmiersprache gewählt. Oftmals wird Kubernetes mit k8s abgekürzt, wobei die “8”
hierbei für die Anzahl der Buchstaben zwischen “K” und “s” steht.

Kubernetes hat sich mit der Zeit gegen andere Container Orchestrator durchgesetzt
und wird aktuell als der Standard für Container-Orchestration angesehen. Wie bereits
erwähnt wurde ein eigenes Container Runtime Interface (CRI) definiert, durch welches
Kubernetes alle größeren Container Runtimes unterstützt. Außerdem haben alle großen
Cloud-Anbieter wie Amazon oder Google Kubernetes-Support. Oftmals bieten diese auch
eigene sogenannte Managed Kubernetes Services an. Hierbei handelt es sich um von
den Cloud-Anbietern eigens entwickelte Schnittstellen für das Arbeiten mit Kubernetes,
welche dieses oftmals erleichtern und für die jeweilige Cloud-Plattform zugeschnitten
sind.

#### 4.2.2 Docker Swarm


Docker beinhaltet mit Docker Swarm, oftmals auch als Swarm Mode bezeichnet, ein haus-
eigenes Orchestrierungstool. Es ist Teil der Docker Engine und ist direkt in die Docker
CLI eingebunden. Man benötigt somit keine zusätzliche Orchestrierungssoftware, wenn
man bereits Docker installiert hat [16].

Docker Swarm ist verglichen mit Kubernetes deutlich einfacher zu bedienen. Kuber-
netes ist viel komplexer und bietet mehr Konfigurationsmöglichkeiten. Trotzdem sind
die Grundfunktionalitäten eines Container Orchestrators alle in Docker Swarm enthalten.
Docker bietet somit einen leicht zu bedienden Container Orchestrator und vereinfacht
die Orchetration für den Nutzer, wenn die Komplexität von Kubernetes nicht benötigt
wird. Docker Swarm beinhaltet beispielsweise von Haus aus kein Monitoring seiner
Services, während Kubernetes dies automatisch macht. Außerdem ist Docker Swarm ist
nur mit Docker-Containern kompatibel, wobei Kubernetes neben Docker auch andere
Container-Formate untersützt.

#### 4.2.3 Docker Compose

Docker bietet neben dem Swarm Mode mit Docker Compose ein weiteres Tool, über
welches man Applikationen bestehend aus mehreren Containern realisieren kann. Hierbei
handelt es sich aber nicht um eine vollwertigen Container Orchestrator. Man kann mit
Docker Compose mehrere Container über ein virtuelles Netzwerk miteinander verbinden
und diese somit miteinander kommunizieren lassen. Es ist aber keine Verteilung auf
mehrere Nodes möglich. Dafür ist die Konfiguration von Docker Compose sehr einfach.
Es eignet sich somit beispielsweise für das lokale Testen einer Cloud Anwendung.

#### 4.2.4 Fleet

Fleet ist ein ehemaliges Projekt von CoreOS. Es wurde in Go entwickelt und unter einer
Open Source Lizenz veröffentlicht [20]. Fleet basiert auf systemd und etcd, einem Key-
Value-Store für verteilte Systeme [18]. Mittlerweile wurde das Projekt eingestellt und
CoreOS empfiehlt Kubernetes als Container Orchestrator [20].

#### 4.2.5 Apache Mesos

Apache Mesos ist ein Open Source Cluster-Manager. Im Gegensatz zu den anderen Orche-
stratoren kann Apache Mesos nicht nur Container orchestrieren, sondern kann generell
für das Verwalten von Anwendungen auf einem Cluster eingesetzt werden. Das Fra-
mework Marathon ist Teil von Apache Mesos und ist für die Container-Orchestration
zuständig. Marathon ist in Scala geschrieben und bietet eine REST-Schnittstelle, über
welche das Cluster verwaltet werden kann [31]. Es werden Container in einem eigenen
Mesos-Container-Format sowie Docker-Container unterstützt [33].

### 4.3 Cloud-Anbieter und Managed Kubernetes Services

Besitzt man selbst keine große Serverinfrastruktur, so bietet es sich an, für seine Cloud-
Anwendungen auf Drittanbieter zurückzugreifen. Diese stellen ihre Hardwareressourcen
als Service zur Verfügung und ermöglichen das Deployment von Cloud-Anwendungen
auf deren Servern. Ein Cloud-Anbieter hat meist ein großes, verteiltes Netzwerk von
Servern. Diese können von Kunden beliebig verwendet werden, um ihre Anwendungen
zu deployen. Dabei wird die Anzahl der gemieteten Ressourcen meist automatisch an
die Bedürfnisse der Kunden angepasst. Benötigt die Anwendung eines Kunden zum
aktuellen Zeitpunkt gerade wenig Ressourcen, so werden auch nur die benötigten Res-
sourcen abgerechnet, wodurch der Kunde nie für unnötig viele Ressourcen zahlen muss.
Außerdem können die Ressourcen flexibel angepasst werden, wodurch automatisch mehr
Ressourcen zur Verfügung gestellt werden, wenn eine Anwendung diese benötigt. All
das wird mit Hilfe von Container-Orchestratoren realisiert.

Zu den bekanntesten Cloud-Anbietern zählt Google mit ihrer Google-Cloud, Amazon
mit Amazon Web Services (AWS), Microsoft mit Azure und IBM mit der IBM Cloud. Sie
alle unterstützen Kubernetes als Container-Orchestrator, bieten hierfür aber jeweils noch
eigene Managed Kubernetes Services an.

Ein Managed Kubernetes Service erleichtert das Arbeiten mit Kubernetes und bietet
dem Kunden eine vereinfachte Schnittstelle zu seinem Kubernetes-Cluster. Diese Schnitt-
stelle ist meist auf den jeweiligen Cloud-Anbieter zugeschnitten und bietet abhängig von
diesem unterschiedliche Funktionalitäten. Oftmals werden hier beispielsweise kompli-
ziertere Konfigurationen automatisiert, wodurch das Aufsetzen eines Clusters vereinfacht
wird. Fast alle großen Cloudanbieter bieten eigene Managed Kubernetes Services an, so
hat Google für ihre Cloud beispielsweise die Google Kubernetes Engine (GKE), Amazon
bietet für AWS Kunden den Amazon Elastic Kubernetes Service (EKS) an, Microsoft hat
den Azure Kubernetes Service (AKS) und IBMs Service heißt IBM Cloud Kubernetes
Service.

### 4.4 Nutzung in der Industrie

Es gibt in der Industrie viele Anwendungsfälle für Cloud-Native-Applikationen. Grund-
legend kann man sagen, dass eine Anwendung immer für die Cloud geeignet ist, wenn
sich ihre Anforderungen mit den typischen Anforderungen von Cloud-Applikationen
überdecken. Dabei handelt es sich also um Applikationen, die unter anderem einen
großen Wert auf Resilienz, Skalierbarkeit und Elastizität legen.

Dies trifft besonders häufig auf Online-Dienste zu, wie beispielsweise Webseiten. So



4.4 Nutzung in der Industrie

sind die meisten größeren Webseiten in der Cloud gehostet. Amazon verwendet bei-
spielsweise ihre eigene Cloud für das Hosten ihres Online-Shop und anderer von ihnen
angebotenen Dienste [2]. Ein weiteres Beispiel wäre Airbnb, welche ebenfalls AWS als
Infrastruktur verwenden [1].

Die weltweit größten Streaminganbieter verwenden ebenfalls Cloud-Technologien für
das Streaming ihrer Filme und Serien. Amazon nutzt für ihr hauseigenes Prime Video
ihre Cloud-Infrastruktur. Auch Netflix zählt zu den bekannteren Kunden von AWS [37].

Hierbei handelt es sich jedoch nur um einige wenige Beispiele für Applikationen, welche
eine Cloud-Technologien verwenden. In der Industrie wird die Cloud für das Deploy-
ment von Anwendungen immer verbreiteter, da Online-Dienste in den letzten Jahren
zunehmend an Beliebtheit und Nutzern gewonnen haben.



## 5 Fallstudie

In den vorherigen Kapiteln wurde auf das Architekturmuster Cloud-Native und des-
sen Eigenschaften eingegangen. Ferner wurden Technologien vorgestellt, welche bei der
Umsetzung einer Cloud-Native-Architektur unterstützen. In diesem Kapitel wird eine
Fallstudie betrachtet, welche eine naive Umsetzung der Cloud-Native-Architektur an-
hand eines beispielhaften Softwareprojektes darstellt. Diese soll im Anschluss evaluiert
werden.

### 5.1 Vorstellung der Fallstudie


Um das Thema "Cloud-Native-Architekturenïnnerhalb eines Beispielprojektes darzustel-
len, wurden zunächst verschiedene Ideen evaluiert. Zu ihnen gehörten unter anderem
ein kleiner Webshop, ein Videokommunkationsservice über WebRTC und ein soziales
Netzwerk zum Teilen von Bildern. Schlussendlich fiel die Wahl auf ein Minimalbeispiel
für eine Online-Wahlplatform. In der Realität müssten eine solche Wahlplatform ihre
Komponenten bei einer großen Menge von Anfragen in der Lage sein, entsprechend zu
skalieren.

### 5.2 Anforderungen

Die Anforderungen an die Fallstudie können in funktionale und nicht funktionale Anfor-
derungen eingeteilt werden. Funktionale Anforderungen beschreiben diejenigen Anforde-
rungen, die zur Funktion des Softwareprojektes unbedingt nötig sind. Nichtfunktionale
Anwendungen legen alle Randbedingungen fest, die zur Qualität der Ausführung der
Software adressiert und umgesetzt werden müssen und über die funktionalen Anforde-
rungen hinaus gehen [quelle: Chris Rupp: Requirements-Engineering und -Management:
Aus der Praxis von klassisch bis agil].

#### 5.2.1 Funktionale Anforderungen

Das durch die Fallstudie entstehende Softwareprojekt hat die Hauptaufgabe, ein Umfrage-
oder Wahlgeschehen zu simulieren. Nutzer sollen in der Lage sein, über eine Webseite
einen aus mehreren Kandidaten auszuwählen und abschließend für ihn eine Stimme ab-
zugeben. Dabei geben sie nicht nur einen Kandidaten, sondern auch ihr Ursprungsland
an. In der Praxis könnten diese Angaben noch um weitere Punkte erweitert werden. Wei-
terhin soll es ihnen möglich sein, die Wahlergebnisse auf einer weiteren Web-Anwendung
zu verfolgen. Das Einsehen von Ergebnissen erfolgt hierbei sowohl nach der Anzahl der
Stimmen für einen Kandidaten, als auch nach den Ergebnissen für einen Kandidaten
pro Abstimmungsort. Zur leichteren Einsicht sollen die Ergebnisse auf letzterer Web-
Anwendung graphisch dargestellt werden.


5 Fallstudie

#### 5.2.2 Nichtfunktionale Anforderungen

Um einige, ausgewählte Anforderungen an eine Cloud-Native-Architektur beispielhaft
darzustellen, sollen zur Umsetzung der Fallstudie entsprechende Technologien gewählt
werden. Hieraus ergeben sich einige Szenarios.

Als zentrale Funktionalität soll es einem Wähler möglich sein, seine Wahl abzugeben.
Diese wird während des Normalbetrieb an die zugehörigen Microservices übermittelt.
Dem Nutzer soll innerhalb weniger Sekunden mitgeteilt werden, ob die Abgabe der
Stimme erfolgreich war.
Ferner gilt es die Ergebnisse einzusehen. Diese werden einem Mitarbeiter bzw. dem
Organisator der Abstimmung über ein weiteres Web-Interface zur Verfügung gestellt.
Während des Normalbetriebs aktualisiert sich das Web-Interface minütlich mit den ak-
tualisierten Wahl-Ergebnissen.
Darüber hinaus soll das System dynamisch skalierbar sein. Hierfür sollen innerhalb
weniger Minuten neue Instanzen der Microservices durch den Container-Orchestrator
gestartet und in das Cluster eingebunden werden können.
Neben den genannten Punkten stellt auch die Ausfall-Sicherheit ein wichtigen Aspekt
der Cloud dar. Dem Nutzer soll dabei stets ein Zugriff auf das Web-Interface der Wähler
und der Wahl-Ergebnisse ermöglicht werden. Die Lade-zeit der Web-Interfaces soll bei
einem Erst-Aufruf durchschnittlich 4 Sekunden betragen. Die maximale Lade-zeit beträgt
8 Sekunden.

### 5.3 Konzeption

Um unsere Wahl-Applikation zu realisieren, müssen die einzelnen Funktionalitäten zu-
nächst in eigene Mikroservices aufgeteilt werden. Wir haben uns dafür entschieden, fünf
bzw. mit Datenbank sechs Microservices zu verwenden.

Die ersten beiden Microservices stellen das Userinterface unserer Anwendung dar. Es gibt
jeweils einen Microservice für das Voting-Frontend (frontend-voting), über welches eine
Stimme abgegeben werden kann, sowie einen Microservice für das Analysis-Frontend
(frontend-analysis), welches das aktuelle Ergebnis der Wahl visualisiert.

Es gibt einen weiteren Microserviceservice-raw-data, welcher für das Speichern und
Abrufen von Daten verantwortlich ist. Dieser ist mit einer Datenbank verbunden, in
unserem Fall einer Redis-Datenbank. Er bietet zwei Funktionalitäten an: das Speichern
einer neu abgegebenen Stimme und das Ausgeben aller bisher erhaltenen Stimmen. Um
die Stimmen nun zu aggregieren, also beispielsweise die Stimmen für ein Land ausgeben
zu lassen, wird ein eigener Microserviceservice-calculateverwendet, welcher für Be-
rechnungen auf den bisher abgegebenen Stimmen zuständig ist. Er erhält die aktuellen
Daten vomservice-raw-data.

Wir haben außerdem noch einen weiteren Microserviceservice-serving-layereinge-
führt, welcher für die Kommunikation zwischen den einzelnen Microservices zuständig
ist. Er bildet somit eine einheitliche Schnittstelle für das Frontend zum "Backend" unserer
Anwendung und koordiniert die einzelnen Anfragen zwischen den einzelnen Microser-
vices.

Unsere Anwendung besteht somit aus den folgenden Komponenten:


### 5.4 Umsetzung


Abbildung 5.1: Architektur Microservices

Wir haben uns für diese Architektur entschieden, da somit jeder Microservice abge-
sehen von der Datenbank stateless ist. Somit können alle Microservices beliebig skaliert
werden.

Wenn nun eine Stimme abgegeben wird, wird von frontend-voting die Stimme zu
service-serving-layergesendet, welcher sie weiter anservice-raw-datareicht, welcher
sie dann anschließend in der Datenbank speichert.
Wird ein Ergebnis der Wahl abgefragt, so sendetfrontend-analysisdiese Anfrage zu
service-serving-layer, welcher die Anfrage zuservice-calculateweiter reicht. Dieser
benötigt nun alle bisher abgegebenen Stimmen, um ein Ergebnis zu berechnen. Hierfür
frägt er wiederumservice-serving-layeran, welcher die Anfrage anservice-raw-data
weiter reicht. Dieser gibt alle bisher abgegebenen Stimmen zurück, anhand dererservice
-calculatedann das Ergebnis berechnet und zurück gibt.

### 5.4 Umsetzung

Da die einzelnen Microservices voneinander unabhängig sind, und in der Praxis oft von
unterschiedlichen Teams entwickelt werden, haben wir uns um dies zu veranschaulichen
dafür entschieden, für unsere Microservices verschiedene Programmiersprachen zu ver-
wenden.

Für den Servicefrontend-votingwurde das JavaScript Framework Vue.js verwendet,
währendfrontend-analysisin Dart geschrieben ist. Fürservice-serving-layerwurde
die Programmiersprache Rust eingesetzt. Die beiden anderen Services,service-raw-data
undservice-calculatesind beide in Go geschrieben.


Für das Erstellen unserer Container-Images haben wir Docker verwendet, da es sich
hierbei um das bekannteste Tool in diesem Bereich handelt und es mit Hilfe von Dockerfi-
les eine sehr einfache Möglichkeit bietet, Container-Images zu erstellen. Zusätzlich bietet
Docker bereits viele Images für bekannte IT-Projekte, wodurch wir direkt ein fertiges
Docker-Image für unsere Redis-Datenbank verwenden konnten. Außerdem haben wir
DockerHub für das Austauschen unserer Images eingesetzt.

Als Container-Orchestrator haben wir Kubernetes eingesetzt, da es sich auch hier um


5 Fallstudie

den Industriestandard handelt. Da wir kein Cluster mit mehreren Nodes zur Verfügung
hatten, um unsere Anwendung zu deployen, haben wir hierfür Minikube verwendet. Mi-
nikube bietet die Möglichkeit, ein Kubernetes-Cluster mit einem einzigen Node innerhalb
einer VM aufzusetzen [35]. Es ist somit gut dafür geeignet, Cloud-Native-Anwendungen
lokal auf einem Rechner auszuführen und zu testen.

Jeder unserer Stateless-Microservices wird in Kubernetes als einzelnes Deployment
abgebildet und kann somit beliebig oft dupliziert werden. Dadurch können die einzelnen
Microservices beliebig skaliert werden. Dies trifft jedoch nicht auf unsere Datenbank
zu. Wir haben uns wegen der ansonsten stark erhöhten Komplexität dafür entschieden,
unsere Datenbank nicht skalierbar sowie nicht persistent zu machen. Es existiert somit
immer nur ein Pod unserer Datenbank und diese kann in unserem Fallbeispiel nicht
skaliert werden.

### 5.5 Evaluation

Abschließend gilt es zu betrachten, auf welche Probleme wir während der Umsetzung
gestoßen sind. Hier muss angemerkt werden, dass die Umsetzung in einem engen Zeit-
rahmen statt gefunden hat. Jedes der gezeigten Konzepte kann noch weiter ausgearbeitet
werden. Ferner wird jedes der gezeigten Konzepte exponentiell komplexer, sollte die
Anwendung in einer produktiven Umgebung eingesetzt werden. Einige dieser Konzepte
werden in den folgenden Abschnitten erläutert. Zunächst gilt es allerdings zu analysieren,
was unsere Anwendung bereits im aktuellen Zustand problemlos umsetzen kann.

Im Kapitel Konzeption wurden die einzelnen Komponenten unser Anwendung gezeigt.
Jede dieser Komponenten stellt ein eigenes Kubernetes Deployment dar. Dies ermöglicht
es jede dieser Komponenten zu skalieren.
Die Kommunikation der Komponenten untereinander ist weiterhin problemlos möglich.
Ferner werden Nutzeranfragen auf die verschiedenen Instanzen der Benutzerschnittstel-
len verteilt. Von außen ist nicht ersichtlich, dass mehrere Instanzen der Komponenten
betrieben werden.
Wurde das Cluster in unserer lokalen Minikube Instanz betrieben, war es möglich beide
Web-Interfaces innerhalb weniger Sekunden zu öffnen. Eine Rückmeldung nach Abga-
be der Stimme erhielt der Nutzer ohne Verzögerung. Ein starten und einbinden einer
neuen Service-Instanz war ebenfalls innerhalb weniger Sekunden möglich. Diese Tests
sind allerdings keinesfalls realistisch. Sollte die Anwendung für ein produktiv System
entwickelt werden, müsste das Cluster zunächst auf dem gewählten Service-Provider
gestartet werden. Anschließend müssten die zuvor genannten Tests erneut durchgeführt
werden.
Hierbei ergeben sich Einschränkungen und Probleme. Diese werden in den nächsten
Abschnitten näher beleuchtet.

#### 5.5.1 Umgang mit State

Im Kapitel Konzeption wurde bereits klar, dass dercalculate-servicestets denserving-
layer-servicenutzt um denraw-data-service, und damit die Datenbank, zu kontaktieren.
Dercalculate-serviceverwaltet keine Art von Cache.
Dies ist Teil der Bemühung die Komponenten weitestgehend stateless zu halten. Eine
Skalierung von stateless Komponenten ist problemlos möglich. Die einzelnen Instanzen
müssen in diesem Fall keine Informationen untereinander teilen und konsistent halten.



5.5 Evaluation

Letzteres reduziert die Komplexität der Anwendung.
Um dennoch stateful Deployments anzulegen stellt Kubernetes Konzepte wieStateful-Sets
[48] zur Verfügung. Im Rahmen dieser Fallstudie wurdenStateful-Setsnicht betrachtet.

#### 5.5.2 Die Datenbank

Im Rahmen dieser Umsetzung wurde eine Redis Datenbank verwendet. Diese ist von
Natur aus stateful. Im vorherigen Abschnitt wurde bereits erläutert, warum dies eine Her-
ausforderung darstellt. Im Rahmen unserer Fallstudie wird die Datenbank nicht skaliert.
Sie befindet sich in einem einzigen Deployment mit einem einzigen Pod. Dies ist natürlich
in einem produktiv Environment nicht praktikabel.
Unserer Umsetzung kann als Abwandlung eines Shared-Repositorys gesehen werden.
Derraw-data-servicestellt dabei das Shared-Repository da. Der Redis-Pod den persis-
tenten Speicher. Ein Nachteil dieses Vorgehens ist die fehlende Skalierbarkeit und die
Datenbank als Single-Point-of-Failure.
Redis bietet allerdings ein Master/Slave Modell an. Dieses nutzt sogenannte Hash-Slots.
Jede der Redis Instanzen verwaltet dabei eine abgeschlossene Menge dieser Hash-Slots.
Die Daten werden also effektiv unter den Redis-Instanzen verteilt. In der Umsetzung der
Fallstudie wurde dieses Konzept nicht genutzt. [41]
Darüber hinaus muss angemerkt werden, dass die Datenbank auch nicht persistiert wird.
Ein Neustart des Clusters führt zur Erstellung neuer Pods. Folglich wird auch eine neue
Instanz der Datenbank angelegt. Auch hierfür wären Stateful-Sets [48] ein Lösungsansatz.

#### 5.5.3 Kommunikations-Logik

Jeder der Micro-Services geht davon aus, dass die jeweils anderen Micro-Services erreich-
bar sind. Dabei wurde nur minimales Error-Handling umgesetzt. In einer Produktiv-
Umgebung gilt es unter anderem eine zentrale Retry-Logik umzusetzen.
Ferner wird die Kommunikation nicht abgesichert. Jede Kommunikation innerhalb des
Clusters und außerhalb des Clusters findet über reines HTTP statt. Ein mitlesen der Kom-
munikation ist damit problemlos möglich.
Darüber hinaus kann jeder Micro-Service die Kommunikation der anderen Micro-Services
einsehen. Um diese Probleme zu lösen eignet sich ein Service-Mesh, wie zum Beispiel
Istio [25].


5 Fallstudie

### 5.6 Fazit

Zusammenfassend lässt sich sagen, dass viele Ressourcen für die Umsetzung einer Cloud-
Native-Anwendung zur Verfügung stehen. Dennoch sind einige Herausforderungen zu
überwinden.
Hierzu zählt nicht zuletzt die Entwicklung für verteilte Systeme, welche selbst Anfor-
derungen mit sich bringt. Darüber hinaus muss sich der Entwickler einer Cloud native
Anwendung in eine Vielzahl von verschiedenen Technologien einarbeiten. Hierfür muss
vorab genügend Zeit eingeplant werden.
Innerhalb der Fallstudie haben wir uns auf Micro-Services und die Container-Orchestration
konzentriert. Damit gelang es uns diese Cloud-Native-Technologien anhand einer Voting-
Applikation zu demonstrieren. Letztere wurde in klar abgetrennte Micro-Services unter-
teilt und wird dynamisch durch den Container-Orchestrator Kubernetes verwaltet.
Dadurch ist es möglich die Anwendung zu skalieren und Anfragen an verschiede-
ne Instanzen der Micro-Services weiterzuleiten. Einige Konzepte der Cloud-Native-
Entwicklung wurden dabei bewusst nicht gelöst. Ein Beispiel hierfür wäre die Persistie-
rung und Orchestrierung der Datenbank.


## Literatur

```
[1] Airbnb - AWS Case Study. en. März 2021.U R L:https://aws.amazon.com/de/
solutions/case-studies/amazon-fulfillment-aurora/.
[2] Amazon - AWS Case Study. en. März 2021.U R L:https://aws.amazon.com/de/
solutions/case-studies/amazon-fulfillment-aurora/.
[3] M.G. Avram. „Advantages and Challenges of Adopting Cloud Computing from
an Enterprise Perspective“. en. In:Procedia Technology12 (2014), S. 529–534.I S S N:
22120173.D O I:10.1016/j.protcy.2013.12.525.U R L:https://linkinghub.
elsevier.com/retrieve/pii/S221201731300710X(besucht am 19. 02. 2021).
[4] cgroups Linux MAN Page. en. März 2021.U R L:https://man7.org/linux/man-
pages/man7/cgroups.7.html.
[5] Chris Richardson.Microservices Pattern: Microservice Architecture pattern.U R L:http:
//microservices.io/patterns/microservices.html(besucht am 22. 02. 2021).
[6] Cloud Native Computing Foundation.cncf/toc. en.U R L:https://github.com/
cncf/toc(besucht am 22. 02. 2021).
[7] CNCF.New Cloud Native Computing Foundation to drive alignment among container
technologies. en-US. Juni 2015.U R L:https://www.cncf.io/announcements/2015/
06/21/new-cloud-native-computing-foundation-to-drive-alignment-among-
container-technologies/(besucht am 22. 02. 2021).
[8] CodingWithNana.Istio & Service Mesh - simply explained in 15 mins - YouTube. de-DE.
U R L:https://www.youtube.com/(besucht am 22. 02. 2021).
[9] containerd GitHub. en. März 2021. U R L:https : / / github. com / containerd /
containerd.

[10] CRI in Kubernetes. en. März 2021.U R L:https://kubernetes.io/blog/2016/12/
container-runtime-interface-cri-in-kubernetes/.

[11] CRI-O. en. März 2021.U R L:https://cri-o.io.

[12] crun GitHub. en. März 2021.U R L:https://github.com/containers/crun.

[13] Rajiv Kumar Ranjan Dimpi Rani. „A Comparative Study of SaaS, PaaS and IaaS
in Cloud Computing“. en. In:International Journal of Advanced Research in Computer
Science and Software Engineering6.4 (Juni 2014).I S S N: 22776451, 2277128X.

[14] Docker.What is a Container? | App Containerization | Docker. en.U R L:https://www.
docker.com/resources/what-container(besucht am 22. 02. 2021).

[15] DockerHub. en. März 2021.U R L:https://hub.docker.com/.

[16] DockerSwarm. en. März 2021.U R L:https://docs.docker.com/engine/swarm/.

[17] Jean Jaques Dubray.Understanding the Costs of Versioning an API (or a Service). Nov.
2013.U R L:https://web.archive.org/web/20150907204623/http://www.ebpml.
org/blog2/index.php/2013/11/25/understanding-the-costs-of-versioning
(besucht am 22. 03. 2021).

[18] etcd GitHub. en. März 2021.U R L:https://github.com/etcd-io/etcd.

[19] Evan Baker.A Comprehensive Container Runtime Comparison. en. Feb. 2021.U R L:
https://www.capitalone.com/tech/cloud/container-runtime/.

[20] Fleet GitHub. en. März 2021.U R L:https://github.com/coreos/fleet.

[21] Brian Foote und Joseph Yoder. „Big Ball of Mud“. In: Sep. 1997.U R L:http://www.
laputan.org/mud/(besucht am 22. 03. 2021).

[22] D. Gannon, R. Barga und N. Sundaresan. „Cloud-Native Applications“. In:IEEE
Cloud Computing4.5 (Sep. 2017). Conference Name: IEEE Cloud Computing, S. 16–
21.I S S N: 2325-6095.D O I:10.1109/MCC.2017.4250939.

[23] Google Workspace (ehemals G Suite): Businesstools für eine effiziente Zusammenarbeit. de.
U R L:https://workspace.google.com/intl/de/(besucht am 15. 02. 2021).

[24] Watts S. Humphrey. „Why Every Business Is a Software Business“. In:Winning
with Software: An Executive Strategy. Feb. 2002.U R L:https://www.informit.com/
articles/article.aspx?p=25491(besucht am 13. 02. 2021).

[25] Istio.Istio. en.U R L:/latest/(besucht am 22. 02. 2021).

[26] julianbrowne.Brewer’s CAP Theorem. Blog.U R L:https://www.julianbrowne.com/
article/brewers-cap-theorem(besucht am 19. 02. 2021).

[27] Kubernetes. en. März 2021.U R L:https://kubernetes.io/de/docs/concepts/
overview/what-is-kubernetes/.

[28] Kubernetes.Production-Grade Container Orchestration. en.U R L:https://kubernetes.
io/(besucht am 22. 02. 2021).

[29] LMCTFY GitHub. en. März 2021.U R L:https://github.com/google/lmctfy.

[30] LXC Gitub. en. März 2021.U R L:https://github.com/lxc/lxc.

[31] Marathon GitHub. en. März 2021. U R L:https : / / github. com / mesosphere /
marathon.

[32] Peter Mell und Tim Grance.The NIST Definition of Cloud Computing. en. Techn. Ber.
NIST Special Publication (SP) 800-145. National Institute of Standards und Tech-
nology, Sep. 2011.D O I:https://doi.org/10.6028/NIST.SP.800- 145.U R L:
https://csrc.nist.gov/publications/detail/sp/800-145/final(besucht am

13. 02. 2021).

[33] Mesosphere Marathon. en. März 2021. U R L:http : / / mesosphere. github. io /
marathon/.

[34] Microsoft.Public Cloud vs Private Cloud vs Hybrid Cloud | Microsoft Azure. en.U R L:
https://azure.microsoft.com/en-us/overview/what-are-private-public-
hybrid-clouds/(besucht am 15. 02. 2021).

[35] Minikube. en. März 2021. U R L:https : / / kubernetes. io / de / docs / setup /
minikube/.

[36] MuleSoft.Microservices vs Monolithic architecture. en.U R L:https://www.mulesoft.
com/resources/api/microservices-vs-monolithic(besucht am 07. 03. 2021).

[37] Netflix - AWS Case Study. en. März 2021.U R L:https://aws.amazon.com/de/
solutions/case-studies/netflix/.

[38] Office 365-Anmeldung | Microsoft Office.U R L:https://www.office.com/(besucht
am 15. 02. 2021).

[39] On Premise vs Cloud. en-GB. Nov. 2020.U R L:https://www.ebcgroup.co.uk/news-
insights/on-premises-vs-cloud(besucht am 19. 02. 2021).


```
Literatur
```
[40] Podman GitHub. en. März 2021.U R L:https://github.com/containers/podman.

[41] Redis cluster tutorial – Redis.U R L:https://redis.io/topics/cluster-tutorial
(besucht am 16. 03. 2021).

[42] Chris Richardson.Microservices Pattern: Monolithic Architecture pattern.U R L:http:
//microservices.io/patterns/monolithic.html(besucht am 06. 03. 2021).

[43] rkt GitHub. en. März 2021.U R L:https://github.com/rkt/rkt.

[44] Boris Scholl, Trent Swanson und Peter Jausovec.Cloud native: using containers, func-
tions, and data to build next-generation applications. English. OCLC: 1130770346. 2019.
I S B N: 978-1-4920-5382-8.

[45] Simon Lohmann.IaaS: Was ist Infrastructure as a Service? - Die moderne Data-Center-
Plattform. de.U R L:https://www.computerwoche.de/a/was-ist-infrastructure-
as- a- service- die- moderne- data- center- plattform, 3332268(besucht am

15. 02. 2021).

[46] Simon Lohmann. Platform as a Service: Was ist PaaS? - Softwareentwicklung in
der Cloud. de. U R L:https : / / www. computerwoche. de / a / was - ist - paas -
softwareentwicklung-in-der-cloud,3332264(besucht am 15. 02. 2021).

[47] S K Sowmya, P Deepika und J Naren. „Layers of Cloud – IaaS, PaaS and SaaS: A
Survey“. en. In: 5 (2014), S. 4.

[48] StatefulSets. en. U R L:https : / / kubernetes. io / docs / concepts / workloads /
controllers/statefulset/(besucht am 16. 03. 2021).

[49] Tom Grey.5 principles for cloud-native architecture—what it is and how to master it. en.
U R L:https://cloud.google.com/blog/products/application-development/
5-principles-for-cloud-native-architecture-what-it-is-and-how-to-
master-it/(besucht am 22. 02. 2021).

[50] Wolfgang Herrmann.SaaS: Was ist Software as a Service?de.U R L:https://www.
computerwoche.de/a/was-ist-software-as-a-service,3332266(besucht am 15. 02. 2021).

[51] Matt Zwicker. SaaS vs On-Premise vs Off-Premise. en. U R L: https : / / info.
iointegration. com / blog / saas - vs - on - premise - vs - off - premise(besucht
am 15. 02. 2021).
```



## Abbildungsverzeichnis

```
2.1 Beispielhafter Aufbau einer Micro-Service Architektur [5].......... 9
2.2 Service-Meshes am Beispiel Istio [8]...................... 13
5.1 Architektur Microservices............................ 29
```

