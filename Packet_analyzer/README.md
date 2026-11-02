# Deep Packet Inspection (DPI) Engine

A C++17-based Deep Packet Inspection engine designed to analyze network traffic, identify applications, extract domain information from HTTP and TLS traffic, and filter packets using configurable blocking rules. The system supports both single-threaded and multithreaded processing architectures.

## Project Overview

Deep Packet Inspection (DPI) is a network traffic analysis technique that examines packet headers and payloads to identify protocols, applications, and communication patterns.

This project implements a DPI engine that processes network traffic stored in PCAP files, parses network protocols, identifies applications using domain information, applies configurable traffic-blocking rules, and generates filtered PCAP files along with detailed traffic analysis reports.

The implementation includes two processing approaches:
- **Single-threaded DPI:** A straightforward packet-processing implementation for learning and analyzing smaller traffic captures.
- **Multithreaded DPI:** A parallel processing architecture designed to distribute packet-processing workloads across multiple threads.

## Key Features

- **PCAP File Processing:** Reads and analyzes network traffic captured in PCAP format.
- **Network Protocol Parsing:** Extracts Ethernet, IPv4, TCP, and UDP header information.
- **Application Identification:** Classifies network traffic using domain names and application signatures.
- **TLS SNI Extraction:** Extracts Server Name Indication (SNI) information from TLS Client Hello messages.
- **HTTP Host Extraction:** Identifies requested domains from HTTP Host headers.
- **Rule-Based Traffic Filtering:** Blocks traffic based on source IP addresses, application types, and domain names.
- **Flow Tracking:** Uses five-tuple connection identification to maintain connection-level state.
- **Multithreaded Processing:** Implements load balancers, fast-path workers, and thread-safe queues.
- **Traffic Reporting:** Generates packet statistics, application breakdowns, and forwarded/dropped packet counts.
- **Filtered PCAP Generation:** Writes permitted packets into an output PCAP file.

## System Architecture

### Single-Threaded Architecture

The single-threaded implementation processes packets sequentially through the following stages:

1. Read packets from the input PCAP file.
2. Parse Ethernet, IP, TCP, and UDP headers.
3. Identify network flows using the five-tuple.
4. Extract application information using HTTP Host and TLS SNI.
5. Evaluate configured blocking rules.
6. Forward permitted packets to the output PCAP file.
7. Generate traffic analysis reports.

### Multithreaded Architecture

The multithreaded implementation distributes packet processing across multiple threads.

The architecture consists of:

- **Reader Thread:** Reads packets from the input PCAP file and distributes them.
- **Load Balancer Threads:** Distribute packets among fast-path workers using flow-based hashing.
- **Fast Path Threads:** Perform packet parsing, application classification, flow tracking, and rule evaluation.
- **Output Writer Thread:** Writes forwarded packets into the output PCAP file.
- **Thread-Safe Queues:** Coordinate communication between processing stages using synchronization mechanisms.

Consistent hashing ensures that packets belonging to the same network flow are assigned to the same processing thread, maintaining flow-state consistency.

## Technology Stack

| Category | Technologies |
|---|---|
| Programming Language | C++17 |
| Networking | Ethernet, IPv4, TCP, UDP |
| Traffic Capture | PCAP |
| Application Identification | HTTP Host, TLS SNI |
| Concurrency | C++ Threads |
| Synchronization | Mutexes, Condition Variables |
| Data Structures | Hash Maps, Queues |
| Testing | Python, Sample PCAP Files |
| Compiler | GCC / Clang |

## Project Structure

```text
Deep-Packet-Inspection-Engine/
│
├── include/
│   ├── pcap_reader.h
│   ├── packet_parser.h
│   ├── sni_extractor.h
│   ├── types.h
│   ├── rule_manager.h
│   ├── connection_tracker.h
│   ├── load_balancer.h
│   ├── fast_path.h
│   ├── thread_safe_queue.h
│   └── dpi_engine.h
│
├── src/
│   ├── pcap_reader.cpp
│   ├── packet_parser.cpp
│   ├── sni_extractor.cpp
│   ├── types.cpp
│   ├── main_working.cpp
│   └── dpi_mt.cpp
│
├── generate_test_pcap.py
├── test_dpi.pcap
└── README.md
```

*Note: The structure represents the documented project architecture. Adjust it to match the actual files in your repository.*

## Building and Running

### Prerequisites

- macOS or Linux
- C++17-compatible compiler (GCC or Clang)
- Python 3 for generating test PCAP files

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR-USERNAME/Deep-Packet-Inspection-Engine.git
cd Deep-Packet-Inspection-Engine
```

### 2. Build the Single-Threaded Version

```bash
g++ -std=c++17 -O2 -I include -o dpi_simple \
src/main_working.cpp \
src/pcap_reader.cpp \
src/packet_parser.cpp \
src/sni_extractor.cpp \
src/types.cpp
```

### 3. Build the Multithreaded Version

```bash
g++ -std=c++17 -pthread -O2 -I include -o dpi_engine \
src/dpi_mt.cpp \
src/pcap_reader.cpp \
src/packet_parser.cpp \
src/sni_extractor.cpp \
src/types.cpp
```

### 4. Generate Test Traffic

```bash
python3 generate_test_pcap.py
```

This generates a sample PCAP file named `test_dpi.pcap`.

### 5. Run the DPI Engine

```bash
./dpi_engine test_dpi.pcap output.pcap
```

### 6. Run with Traffic-Blocking Rules

```bash
./dpi_engine test_dpi.pcap output.pcap \
--block-app YouTube \
--block-app TikTok \
--block-ip 192.168.1.50 \
--block-domain facebook
```

### 7. Configure Multithreaded Processing

```bash
./dpi_engine input.pcap output.pcap --lbs 4 --fps 4
```

This configures four load balancer threads and four fast-path threads per load balancer, according to the documented architecture.

## Application Identification and Blocking

The DPI engine uses domain information extracted from network traffic to classify applications.

Examples of supported application categories include:

- YouTube
- Facebook
- Google
- GitHub
- DNS
- HTTP
- HTTPS

Traffic can be filtered using three types of rules:

| Rule Type | Example | Function |
|---|---|---|
| IP Blocking | 192.168.1.50 | Blocks traffic from a specified source IP |
| Application Blocking | YouTube | Blocks identified application traffic |
| Domain Blocking | facebook | Blocks domains matching a configured pattern |

Once a connection is identified as blocked, subsequent packets belonging to that flow are dropped.

## Performance and Reporting

After processing, the engine generates a report containing:

- Total packets and bytes processed.
- TCP and UDP packet counts.
- Forwarded and dropped packet counts.
- Application-wise traffic distribution.
- Detected domain names and TLS SNI values.
- Load balancer and fast-path thread statistics.

## Learning Outcomes

This project demonstrates practical implementation of:

- Network protocol analysis and packet parsing.
- Deep Packet Inspection and application classification.
- TLS handshake and SNI extraction.
- Five-tuple-based connection tracking.
- Rule-based network traffic filtering.
- Multithreaded programming and synchronization.
- Producer-consumer architecture.
- PCAP file processing and traffic analysis.

## Future Enhancements

Potential improvements include:

- Additional application signatures.
- Bandwidth throttling.
- Real-time traffic monitoring dashboard.
- QUIC and HTTP/3 support.
- Persistent traffic-filtering rules.

## Contributors

Developed as a networking and systems programming project.

## License

Refer to the applicable license before redistributing or modifying this project.
