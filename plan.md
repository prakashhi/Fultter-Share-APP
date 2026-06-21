# 🗺️ Master Clean Architecture & Verification-Driven Plan (`plan.md`)

This document outlines the absolute, complete, and production-ready architectural plan and implementation roadmap for the high-speed, secure, offline file-sharing Flutter application.

---

## 🚀 1. Core Objectives & Speed Metrics
* **Technology:** Raw TCP/TLS Sockets over a local Wi-Fi Hotspot for the absolute fastest hardware-level data transfer.
* **Encryption:** End-to-end TLS encryption via dynamically generated self-signed certificates.
* **Multi-Device Support (One-to-Many):** Parallel binary chunk streaming from one sender to multiple secure receivers with backpressure management.
* **On-the-Fly Folder Transfers:** Directory streaming via high-performance TAR format without generating a heavy temporary ZIP/TAR file on disk.

---

## 📂 2. Absolute File Structure Blueprint
Every single file must reside in its designated layer according to Clean Architecture principles.

```text
lib/
├── main.dart                          # App entry point & Dependency Injection configuration
│
├── core/                              # Shared, framework-independent helper utilities
│   ├── constants/
│   │   ├── app_constants.dart         # Core configurations (app metadata, limits)
│   │   ├── network_constants.dart     # TCP Ports, UDP Broadcast, Magic Bytes, buffer sizes
│   │   └── storage_constants.dart     # Database collections, local paths configurations
│   ├── errors/
│   │   ├── failures.dart              # Domain layer failures for Either pattern
│   │   └── exceptions.dart            # Data layer exceptions mapping socket/DB errors
│   ├── theme/
│   │   ├── app_colors.dart            # Brand and semantic app colors
│   │   └── app_theme.dart             # Global application theme (Dark / Light support)
│   ├── utils/
│   │   ├── security_context_helper.dart # Self-signed cert generation & TLS context construction
│   │   ├── tar_streamer.dart          # Recursive directory directory-to-stream compiler
│   │   ├── speed_calculator.dart      # Sliding window throughput calculator (MB/s)
│   │   └── formatters.dart            # Standard formatting (Bytes to MB/GB, time formats)
│   └── di/
│       └── service_locator.dart       # Service container & Riverpod provider overrides
│
├── data/                              # Hardware & Database Implementation Layer
│   ├── datasources/
│   │   ├── local/
│   │   │   ├── history_local_datasource.dart # Isar local DB persistence implementation
│   │   │   └── preferences_datasource.dart   # SharedPreferences persistent key/value store
│   │   └── network/
│   │       ├── tcp_socket_datasource.dart    # Raw SecureSocket & SecureServerSocket control
│   │       ├── discovery_datasource.dart     # UDP Multicast Broadcast scanner
│   │       └── hotspot_datasource.dart       # Platform channels mapping Native Android Hotspot APIs
│   ├── models/
│   │   ├── transfer_history_model.dart # Isar @collection persistence model
│   │   ├── peer_device_model.dart      # JSON mapper class for peer discovery data
│   │   ├── file_manifest_model.dart    # Mapping layer for transfer headers and TCP frames
│   │   └── transfer_progress_model.dart # UI progress state representation object
│   └── repositories/
│       ├── wifi_transfer_repository_impl.dart # Implements ITransferRepository contract
│       └── history_repository_impl.dart       # Implements IHistoryRepository contract
│
├── domain/                            # Pure Business Logic Layer (No Flutter / Plugin imports)
│   ├── entities/
│   │   ├── shareable_file.dart        # Holds paths, size, extension, isFolder flag
│   │   ├── peer_device.dart           # Holds Peer credentials (IP, Port, Fingerprint)
│   │   ├── transfer_session.dart      # Tracks live transfer speed, progress, connected peers
│   │   └── transfer_history.dart      # Holds a past file transfer transaction log
│   ├── repositories/
│   │   ├── i_transfer_repository.dart # Abstract contract for Wifi controls & Socket streams
│   │   └── i_history_repository.dart  # Abstract contract for Isar DB CRUD operations
│   └── usecases/
│       ├── transfer/
│       │   ├── start_hotspot_usecase.dart
│       │   ├── stop_hotspot_usecase.dart
│       │   ├── discover_peers_usecase.dart
│       │   ├── connect_to_peer_usecase.dart
│       │   ├── send_files_usecase.dart
│       │   ├── receive_files_usecase.dart
│       │   └── cancel_transfer_usecase.dart
│       └── history/
│           ├── get_history_usecase.dart
│           ├── clear_history_usecase.dart
│           └── delete_history_item_usecase.dart
│
└── presentation/                      # UI Rendering & State Control Layer
    ├── global_widgets/
    │   ├── custom_progress_bar.dart   # Animated performance progress indicator
    │   ├── speedometer.dart           # Dashboard speedometer widget
    │   ├── file_type_icon.dart        # Type-specific icon selector
    │   └── loading_overlay.dart       # Fullscreen loading barrier widget
    ├── home/
    │   ├── home_page.dart             # Role selection (Send/Receive) Dashboard
    │   └── home_provider.dart         # Home page state notifier (Hotspot configuration)
    ├── qr_connection/
    │   ├── qr_scanner_page.dart       # Camera scanning implementation
    │   ├── qr_display_page.dart       # Displays QR code containing dynamic credentials
    │   └── qr_provider.dart           # Encodes/Decodes dynamic JSON connection models
    ├── file_picker/
    │   ├── file_picker_page.dart      # Standard categorized files selection view
    │   ├── file_category_selector.dart # Category tabs widget
    │   └── file_picker_provider.dart  # Manages queues of selected items
    ├── transfer_screen/
    │   ├── pages/
    │   │   ├── send_transfer_page.dart    # Live screen for sender
    │   │   └── receive_transfer_page.dart # Live screen for receiver
    │   ├── widgets/
    │   │   ├── transfer_progress_card.dart # Overall speed/ETA card
    │   │   ├── file_transfer_tile.dart     # Individual item row progress
    │   │   └── transfer_controls.dart      # Cancel / Abort action buttons
    │   └── transfer_provider.dart     # Orchestrates Riverpod states for active transfers
    └── history/
        ├── history_page.dart          # Display list of past transfers
        ├── history_item_tile.dart     # Individual history log row
        └── history_provider.dart      # Riverpod provider pulling logs from Isar DB
```

---

## ⚙️ 3. Step-by-Step Implementation & Verification Roadmap

### Phase 1: Environment Configuration
* **Task 1.1: Configure Dependencies**
  * **Action:** Update `pubspec.yaml` with state, networking, DB, and UI packages.
  * **Test:** Run `flutter pub get`. Must exit with status code `0`.
* **Task 1.2: Directory Scaffold**
  * **Action:** Construct the full directory layout as specified in the blueprint.
  * **Test:** Run `Get-ChildItem -Recurse lib` to verify directories exist.

### Phase 2: Core Infrastructure
* **Task 2.1: Dynamic TLS Context (`security_context_helper.dart`)**
  * **Action:** Programmatically generate self-signed keys and return a dynamic `SecurityContext`.
  * **Test:** Write `test/core/security_context_test.dart`. Ensure SHA-256 fingerprint extraction succeeds.
* **Task 2.2: Stream-Based Folder Packager (`tar_streamer.dart`)**
  * **Action:** Map directories recursively and construct direct binary streams in TAR format.
  * **Test:** Write `test/core/tar_streamer_test.dart`. Pipe a mock folder into a parser and verify identical directory output.
* **Task 2.3: Sliding Window Throughput Calculator (`speed_calculator.dart`)**
  * **Action:** Record rolling queue data over 1.0 second to calculate MB/s speed.
  * **Test:** Write `test/core/speed_calculator_test.dart`. Seed constant payloads and verify output speed.

### Phase 3: Pure Domain Layer
* **Task 3.1: Construct Domain Entities**
  * **Action:** Code `ShareableFile`, `PeerDevice`, `TransferSession`, `TransferHistory`.
  * **Test:** Run `flutter analyze`. Must complete with zero errors and no dependency leaks.
* **Task 3.2: Write Abstract Interfaces**
  * **Action:** Setup contracts `ITransferRepository` and `IHistoryRepository`.
  * **Test:** Run `flutter analyze`.
* **Task 3.3: Write Domain Use Cases**
  * **Action:** Implement single-responsibility transaction controllers.
  * **Test:** Write `test/domain/usecases_test.dart`. Verify mocks match call signatures.

### Phase 4: Local Storage & Network Layer
* **Task 4.1: History Database (`Isar`)**
  * **Action:** Add entity mappings to `transfer_history_model.dart` and compile schemas.
  * **Test:** Run `flutter pub run build_runner build`. Run local tests in `test/data/isar_test.dart`.
* **Task 4.2: UDP Discovery Datasource**
  * **Action:** Develop heartbeat broadcasts on Port `5555` to find nearest peers.
  * **Test:** Write `test/data/udp_discovery_test.dart`. Broadcast payload and assert discovery succeeds.
* **Task 4.3: Secure One-to-Many TLS Sockets**
  * **Action:** Build a parallel streaming TCP pipeline handling multiple simultaneous clients via secure sockets.
  * **Test:** Write `test/data/secure_transfer_integration_test.dart`. Stream a 5MB payload to 3 mock sockets and assert checksum alignment.

### Phase 5: Presentation & UI Layer
* **Task 5.1: QR Connection Codecs**
  * **Action:** Setup JSON string parsers for profile serialization.
  * **Test:** Write `test/presentation/qr_codec_test.dart`. Assert stringify/parse parity.
* **Task 5.2: Riverpod Transfer Provider**
  * **Action:** Bind transfer streams to the active UI state.
  * **Test:** Write `test/presentation/transfer_provider_test.dart`. Stream data and assert visual state reactivity.
* **Task 5.3: Interface screens**
  * **Action:** Construct visual layouts for Home, Picker, Live Transfer, and History screens.
  * **Test:** Run `flutter run`. Perform manual clicks and verify state changes.

### Phase 6: OS Integration
* **Task 6.1: Android Permissions & Security**
  * **Action:** Inject permissions into `AndroidManifest.xml` and setup dynamic certificate parsing bypasses for trusted SHA-256 signatures.
  * **Test:** Build and execute APK, ensuring no operating system authorization crashes.

---

## 🔄 4. The Loop-Verification Workflow

Every single task must strictly adhere to the loop workflow:
1. Implement the task code.
2. Implement the designated test file inside `/test`.
3. Run the tests.
4. Correct failures until the test passes.
5. Record completion and advance to the next task.
