# Requirements Document

## Introduction

Sistem Laporan Banjir Real-time untuk Warga Malang adalah aplikasi web yang memungkinkan warga melaporkan kondisi banjir secara real-time dengan validasi bukti digital. Sistem ini mengatasi masalah utama informasi banjir yaitu verifikasi keaslian laporan ("beneran atau hoax tahun lalu?"). Aplikasi menggunakan peta interaktif dengan color-coding berdasarkan ketinggian air, agregasi bukti dari media sosial, dan sistem time-stamp kedaluwarsa untuk memastikan informasi tetap akurat dan terkini.

## Glossary

- **Flood_Report_System**: Sistem utama yang mengelola laporan banjir dari warga
- **User**: Warga yang menggunakan aplikasi untuk melaporkan atau melihat kondisi banjir
- **Flood_Report**: Laporan banjir yang berisi lokasi, tingkat ketinggian air, dan bukti digital
- **Water_Level**: Tingkat ketinggian air (Siaga/Kuning, Bahaya/Oranye, Evakuasi/Merah)
- **Digital_Proof**: Bukti berupa foto upload langsung atau link media sosial (Instagram, Twitter/X, TikTok)
- **Safe_Route**: Jalur yang ditandai sebagai kering/aman untuk dilalui
- **Report_Expiry**: Masa berlaku laporan (default 3 jam) sebelum membutuhkan konfirmasi ulang
- **Upvote**: Konfirmasi dari user lain bahwa kondisi banjir masih berlaku
- **CCTV_Camera**: Kamera CCTV publik Kota Malang yang dapat diakses untuk verifikasi visual kondisi jalan
- **CCTV_Stream**: Video streaming dari CCTV menggunakan WebRTC atau HLS protocol

## Requirements

### Requirement 1

**User Story:** Sebagai warga, saya ingin login dengan mudah menggunakan akun sosial media, sehingga saya dapat melaporkan banjir dengan cepat tanpa proses registrasi yang rumit.

#### Acceptance Criteria

1. WHEN a user clicks the login button THEN the Flood_Report_System SHALL display authentication options via Clerk provider
2. WHEN a user successfully authenticates THEN the Flood_Report_System SHALL create a session and redirect to the main map view
3. WHEN a user clicks logout THEN the Flood_Report_System SHALL terminate the session and return to public view

### Requirement 2

**User Story:** Sebagai warga, saya ingin melihat peta interaktif dengan titik-titik banjir berwarna, sehingga saya dapat memahami kondisi banjir di sekitar saya dengan cepat.

#### Acceptance Criteria

1. WHEN the map view loads THEN the Flood_Report_System SHALL display an interactive map centered on Malang city using Leaflet.js
2. WHEN flood reports exist THEN the Flood_Report_System SHALL display markers with color-coding based on Water_Level (Yellow for ankle-level, Orange for knee/waist-level, Red for chest/roof-level)
3. WHEN a user clicks a flood marker THEN the Flood_Report_System SHALL display a popup containing the Digital_Proof (embedded photo or social media preview)
4. WHEN a flood report exceeds Report_Expiry time without confirmation THEN the Flood_Report_System SHALL fade the marker opacity to indicate stale data

### Requirement 3

**User Story:** Sebagai warga, saya ingin melaporkan banjir dengan drop pin di peta dan melampirkan bukti, sehingga laporan saya dapat diverifikasi oleh warga lain.

#### Acceptance Criteria

1. WHEN a logged-in user clicks the "LAPOR" button THEN the Flood_Report_System SHALL enable pin-drop mode on the map
2. WHEN a user drops a pin on the map THEN the Flood_Report_System SHALL display a form to select Water_Level and attach Digital_Proof
3. WHEN a user selects Water_Level THEN the Flood_Report_System SHALL accept one of three options (Siaga, Bahaya, Evakuasi)
4. WHEN a user submits a report with valid location and Water_Level THEN the Flood_Report_System SHALL store the Flood_Report in Supabase and display it on the map immediately via real-time subscription
5. WHEN a user attempts to submit a report without selecting a location THEN the Flood_Report_System SHALL prevent submission and display a validation message

### Requirement 4

**User Story:** Sebagai warga, saya ingin melampirkan bukti dari media sosial dengan paste link saja, sehingga saya tidak perlu upload video yang memakan kuota besar saat darurat.

#### Acceptance Criteria

1. WHEN a user pastes a social media link (Instagram, Twitter/X, TikTok) THEN the Flood_Report_System SHALL validate the URL format
2. WHEN a valid social media link is submitted THEN the Flood_Report_System SHALL store the link and display an embedded preview using oEmbed
3. WHEN a user chooses to upload a photo directly THEN the Flood_Report_System SHALL accept image files and store them in Supabase Storage
4. WHEN displaying Digital_Proof in the popup THEN the Flood_Report_System SHALL render embedded social media content or uploaded image

### Requirement 5

**User Story:** Sebagai warga, saya ingin mengkonfirmasi bahwa banjir masih terjadi di suatu lokasi, sehingga informasi tetap akurat dan terkini.

#### Acceptance Criteria

1. WHEN a user views a flood marker THEN the Flood_Report_System SHALL display a "Masih Banjir" (Upvote) button
2. WHEN a user clicks the Upvote button THEN the Flood_Report_System SHALL reset the Report_Expiry timer and increment the confirmation count
3. WHEN a report receives an Upvote THEN the Flood_Report_System SHALL restore full marker opacity and update the timestamp

### Requirement 6

**User Story:** Sebagai warga yang ingin pulang kerja, saya ingin menandai jalur yang kering/aman, sehingga orang lain tahu rute alternatif yang bisa dilalui.

#### Acceptance Criteria

1. WHEN a logged-in user clicks "Lapor Jalur Aman" button THEN the Flood_Report_System SHALL enable safe route marking mode
2. WHEN a user marks a Safe_Route THEN the Flood_Report_System SHALL display it as a green marker or line on the map
3. WHEN displaying Safe_Route THEN the Flood_Report_System SHALL apply the same Report_Expiry logic as flood reports

### Requirement 7

**User Story:** Sebagai warga, saya ingin memfilter tampilan peta berdasarkan kondisi, sehingga saya dapat fokus pada informasi yang relevan untuk kebutuhan saya.

#### Acceptance Criteria

1. WHEN a user clicks filter "Bisa Lewat Mobil" THEN the Flood_Report_System SHALL display only markers with Water_Level Siaga (Yellow) and Safe_Routes
2. WHEN a user clicks filter "Lumpuh Total" THEN the Flood_Report_System SHALL display only markers with Water_Level Evakuasi (Red)
3. WHEN a user clears filters THEN the Flood_Report_System SHALL display all markers

### Requirement 8

**User Story:** Sebagai warga, saya ingin melihat feed CCTV publik di sekitar lokasi banjir, sehingga saya dapat memverifikasi kondisi jalan secara visual sebelum melewatinya.

#### Acceptance Criteria

1. WHEN the map loads THEN the Flood_Report_System SHALL display CCTV_Camera markers from the cctv_organized.json data on the map
2. WHEN a user clicks a CCTV_Camera marker THEN the Flood_Report_System SHALL display a popup with the camera name and a button to view live stream
3. WHEN a user clicks "Lihat CCTV" button THEN the Flood_Report_System SHALL open a modal displaying the CCTV_Stream using HLS or WebRTC protocol
4. WHEN displaying CCTV markers THEN the Flood_Report_System SHALL use a distinct icon (camera symbol) to differentiate from flood report markers
5. WHEN a flood report is near a CCTV location (within 200 meters) THEN the Flood_Report_System SHALL show a "Cek CCTV Terdekat" link in the flood report popup

### Requirement 9

**User Story:** Sebagai warga, saya ingin tampilan yang mudah digunakan saat panik dan kehujanan, sehingga saya dapat melaporkan atau melihat info banjir dengan cepat.

#### Acceptance Criteria

1. WHEN the application loads THEN the Flood_Report_System SHALL display dark mode by default to conserve battery
2. WHEN displaying the main interface THEN the Flood_Report_System SHALL show a large "LAPOR" button at the bottom of the screen for easy thumb access
3. WHEN displaying UI elements THEN the Flood_Report_System SHALL use high contrast Neobrutalism style for visibility in poor conditions
