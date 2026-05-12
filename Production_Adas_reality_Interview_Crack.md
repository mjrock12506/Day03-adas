# Production ADAS Reality + Interview Crack Guide
### From Academic Toys to Hireable Engineer in 7 Days
**Prepared for: Mridhul Sudhagona — OPT clock running, no time to waste**

---

## Opening Reality Check (Read This First)

You've built three things in three days:
- Day 1: Python sensor analytics with pandas/NumPy
- Day 2: C/C++ AEB on ESP32 with circular buffer
- Day 3: PID, KF, EKF in Python

**What you have:** Working code, the architecture in your head, the vocabulary growing.

**What an interviewer wants:** Evidence that you can ship this code in production, talk about it in business terms, and not break under cross-examination by a senior engineer who's been doing this for 10 years.

The gap is closable. But you have to **reframe** what you built — same code, production language. And you have to **add a few specific pieces** that signal "I get the real world, not just the textbook."

Here it is, all of it.

---

# Part 1 — Every Project You Built, Reframed As Production

When asked "Tell me about a project you worked on," you do NOT say "I built a sliding window filter in pandas." You say what's below.

## Project 1 — Sensor Log Analytics Pipeline (Day 1)

### Production framing (use this exact language)
> "Built a sensor log analysis pipeline for vehicle telemetry data — pandas-based event detection on time-series sensor logs to identify hard-braking events, sharp turns, and high-G maneuvers. Implemented vectorized NumPy operations achieving roughly 100x speedup over equivalent Python loops, simulating the kind of preprocessing required before feeding data into perception model training pipelines. Designed the architecture to scale to fleet-level data — millions of samples per vehicle per day."

### Business case (the angle that wins interviews)
> "Fleet-level log analysis is how Tier-1 suppliers validate ADAS features post-deployment. A single test fleet generates terabytes daily; engineers must detect anomalies — false AEB activations, missed pedestrian detections — across millions of drives. The pandas/NumPy workflow I built is the entry-level analytics layer that scales to Spark/Databricks at fleet scale."

### Real-world tools your work is the foundation for
- **Parquet/Apache Arrow** — columnar storage for fleet logs (your CSV scales to this)
- **DuckDB / Spark** — SQL-on-files at billion-row scale
- **MLflow** — tracking which model version produced which decisions in logs
- **Foxglove Studio** — industry-standard visualizer for sensor logs (replaces matplotlib)
- **MCAP** — Foxglove's binary log format (replaces CSV in production)

### What's missing from your portfolio that interviewers will probe
- "Have you used Foxglove or rosbag?" — answer: "Not yet but I understand the architecture from my pandas-based equivalent"
- "What's MCAP?" — Foxglove's open-source replacement for rosbag2

### Two-line resume bullet
> Built vehicle sensor log analytics pipeline in pandas/NumPy with event detection (hard-braking, sharp-turn) and rolling-window noise filtering; demonstrated 100x speedup via vectorized operations; designed to scale toward Parquet+Spark for fleet-level deployment.

---

## Project 2 — Real-Time AEB on ESP32 (Day 2)

### Production framing
> "Implemented an Automatic Emergency Braking prototype on simulated ESP32 hardware using ultrasonic distance sensing, a fixed-size circular buffer for noise filtering, threshold-based emergency triggering, and a 100Hz control loop benchmarked at 1.3ms cycle time — well within the 10ms real-time budget required for ISO 26262 ASIL-D compliance. The architecture mirrors production AEB systems: perception layer reads sensor data, filtering layer rejects noise, decision layer evaluates safety conditions, actuation layer commands output."

### Business case
> "AEB became mandatory under EU GSR 2024 and is expanding to all new US vehicles. Every Tier-1 supplier has dozens of engineers working on AEB variants — front, rear, pedestrian, cyclist. My demo proves I understand the real-time architecture: pre-allocated buffers, deterministic timing, threshold-based actuator command, no heap allocation in the hot loop."

### What an interviewer might ask (and how to answer)

**Q: "What's the latency budget for AEB?"**
> "From sensor input to actuator command, production AEB targets sub-100ms end-to-end. Within that, the perception layer typically gets ~50ms, fusion gets ~20ms, decision gets ~10ms, and actuator command transmission over CAN takes another 10-20ms. My loop runs at 1.3ms which fits comfortably."

**Q: "Why fixed-size buffer instead of std::vector?"**
> "std::vector triggers heap reallocation on growth, which is non-deterministic. ISO 26262 ASIL-D requires deterministic execution timing. Fixed-size buffers are pre-allocated at startup and never resize at runtime — guaranteed bounded behavior."

**Q: "How would you scale this to a real radar?"**
> "Radar provides not just range but velocity (via Doppler) and target tracking IDs. I'd replace the ultrasonic threshold with a Kalman-filtered Time-To-Collision calculation, fuse with camera for object classification, and implement the production pattern of perception → fusion → decision → actuation. The buffer pattern stays the same."

### Real-world tools this maps to
- **Vector CANoe** — actual development environment for AEB (replaces Wokwi)
- **dSPACE SCALEXIO / NI VeriStand** — HIL test rigs for production AEB
- **Aurix Tricore microcontroller** — what production AEB actually runs on (replaces ESP32)
- **AUTOSAR Classic** — the OS layer (replaces Arduino)
- **CARLA / IPG CarMaker** — simulation environments for full vehicle testing

### Production failure modes your demo doesn't address (be ready to discuss these)
- **Brake actuator failure** — production AEB has redundant brake systems and degraded modes
- **Sensor blindness** — rain, fog, sun glare cause sensor dropouts; production handles this with sensor fusion
- **False positives** — production AEB has ~10⁻⁶ false-positive rate; one false brake per million miles
- **CAN bus saturation** — under heavy load, the brake command might be delayed

### Two-line resume bullet
> Built real-time AEB prototype on ESP32 with HC-SR04 ultrasonic sensing, ring-buffer noise filtering, and threshold-triggered actuation; benchmarked control loop at 1.3ms cycle time vs 10ms ASIL-D budget; demonstrated production-grade pattern of pre-allocated buffers and deterministic execution.

---

## Project 3 — Sensor Fusion: PID + KF + EKF (Day 3)

### Production framing
> "Implemented a complete control and estimation stack: discrete PID controller with Ziegler-Nichols tuning for ACC speed regulation, linear Kalman Filter fusing simulated GPS + IMU data with 5x position-error improvement over IMU dead-reckoning, and Extended Kalman Filter using a 5-state CTRA motion model for curved-path tracking. Demonstrated the standard architecture used in production ADAS localization: high-rate IMU prediction step, low-rate GPS update step, with the EKF Jacobian linearizing nonlinear motion equations at every timestep."

### Business case
> "Sensor fusion is the foundation of every L2+ ADAS system. A car cannot drive itself if it doesn't know with confidence where it is and how fast it's going. GPS alone is too noisy and unavailable in tunnels. IMU alone drifts by tens of meters within a minute. Fusion via Kalman variants is the only path to localization that's accurate enough for L2+ autonomy. Every OEM has a localization team — Bosch, Continental, Aptiv each have hundreds of engineers in this domain."

### What an interviewer will ask (this is the senior-level set)

**Q: "Why EKF instead of UKF?"**
> "EKF is computationally cheaper because Jacobians are sparse for typical motion models like CTRA — most Jacobian entries are zero. UKF propagates a set of sigma points through the nonlinear function instead of linearizing, capturing the true posterior distribution more accurately. UKF is preferred when nonlinearity is severe over a single timestep — heavy maneuvers, large dt, or unmodeled dynamics. For typical ADAS at 100Hz update rates, EKF is the production standard."

**Q: "How do you tune Q and R?"**
> "Two methods. First, from sensor datasheets — R is the squared standard deviation reported by the sensor manufacturer (a Bosch GPS unit might spec 2.5m at 1-sigma; R = 6.25). Q is harder — it captures unmodeled dynamics, so you tune empirically by minimizing innovation covariance variance against logged ground-truth data. Production teams use NIS testing — Normalized Innovation Squared — to validate Q over thousands of recorded drives."

**Q: "What's the difference between a Kalman Filter and a Particle Filter?"**
> "Kalman variants assume Gaussian noise and produce a single Gaussian posterior. Particle Filter represents the posterior with thousands of weighted samples — works for arbitrary non-Gaussian distributions. Particle Filter is dramatically more expensive (1000+ particles per state) and not used in real-time ADAS. It shows up in offline applications like SLAM."

**Q: "What's a factor graph?"**
> "A factor graph is a more flexible alternative to Kalman Filter, particularly popular in robotics. Each measurement and motion constraint becomes a 'factor' connecting state nodes; optimization finds the maximum-likelihood state estimate over all factors. GTSAM and Ceres are the major libraries. Factor graphs handle non-Gaussian distributions, asynchronous sensors, and out-of-order measurements better than EKF. Apollo and Autoware use them heavily for SLAM and localization."

**Q: "What's the smallest position error you've seen in your filter?"**
> "On my simulated test with 5m-sigma GPS noise and 0.08 m/s² IMU bias, the linear Kalman Filter achieved 2.3m mean position error — 5x better than IMU dead-reckoning at 11.7m. Production systems with RTK-corrected GPS (centimeter-level), tactical-grade IMU, and wheel-odometry fusion routinely achieve sub-10cm accuracy."

### Real-world tools and libraries

- **Eigen** — C++ linear algebra library; the production replacement for NumPy. Memorize: `Eigen::Matrix3d`, `Eigen::Vector4f`, `.transpose()`, `.inverse()`. **Every ADAS C++ codebase uses Eigen.**

- **GTSAM (Georgia Tech Smoothing And Mapping)** — Factor graph library used in production SLAM/localization. Apollo's open-source localization is GTSAM-based.

- **Ceres Solver** — Google's nonlinear least-squares optimization library; used in Visual SLAM, calibration, sensor fusion.

- **ROS2 `robot_localization`** — Production-quality EKF/UKF node, multi-sensor capable. Used by hundreds of robotics companies. You can run it on your laptop today.

- **Apollo's `localization` module** — Open-source autonomous driving stack from Baidu. The localization module is a textbook example of production EKF + GNSS-RTK + IMU + map matching.

- **NDT (Normal Distributions Transform)** — Point-cloud-based localization, used by Autoware.

### Production failure modes (have answers ready)

- **GPS outages in tunnels** → fall back to IMU-only with degraded confidence; production systems track HDOP/VDOP from GPS receiver
- **IMU bias drift over temperature** → online bias estimation by augmenting state vector
- **Sensor delay (GPS reports 100ms-old position)** → use OOSM (Out-Of-Sequence Measurement) filtering or rewind-and-replay
- **Filter divergence** → monitor innovation covariance; if it exceeds expected bounds, reinitialize filter
- **Numerical instability in P matrix** → use Joseph form or square-root Kalman Filter

### Two-line resume bullet
> Built discrete PID controller with Ziegler-Nichols tuning, linear Kalman Filter (GPS+IMU fusion, 5x accuracy improvement), and Extended Kalman Filter with CTRA motion model and Jacobian linearization in Python — demonstrating the core localization and control pipeline used in production L2 ADAS systems.

---

# Part 2 — The 30 Production Terms You Must Know Cold

Every interview probes these. Knowing the term ≠ knowing it. You must be able to use it in a sentence.

## Sensor & Perception Terms

| Term | One-line definition for interview |
|---|---|
| **Sensor fusion** | Combining multiple noisy sensor streams into a single best estimate, typically via Kalman variants or factor graphs. |
| **Time synchronization** | Aligning timestamps across sensors running at different rates and with different latencies. Critical for fusion. |
| **PPS (Pulse Per Second)** | Hardware sync signal from GNSS receiver used to timestamp all other sensors precisely. |
| **Extrinsic calibration** | Knowing the rigid transformation (rotation+translation) between sensor coordinate frames. |
| **Intrinsic calibration** | Per-sensor internal parameters (camera focal length, lens distortion, IMU scale factors). |
| **NMEA / RTCM** | GNSS sentence formats. NMEA for position output, RTCM for differential corrections. |
| **RTK GNSS** | Real-Time Kinematic GNSS — centimeter accuracy via base-station differential corrections. |
| **VRU** | Vulnerable Road User (pedestrians, cyclists, motorcyclists). |
| **Free space detection** | Identifying drivable area, not just objects. |

## Control & Estimation Terms

| Term | One-line definition |
|---|---|
| **Closed-loop control** | Feedback-based control where output is measured and used to adjust input. |
| **Open-loop control** | No feedback, command-only. Rare in safety-critical applications. |
| **Bode plot / Phase margin** | Frequency-domain controller stability analysis. |
| **State observer** | A system that estimates internal states from outputs. Kalman Filter is the optimal observer for linear-Gaussian systems. |
| **Observability** | Whether all states can be inferred from available measurements. Critical question before designing any filter. |
| **Innovation** | The difference between predicted and actual measurement (`z - H·x`). Used for divergence detection. |
| **NIS (Normalized Innovation Squared)** | Innovation magnitude scaled by its covariance. NIS testing validates filter consistency. |
| **JPDA (Joint Probabilistic Data Association)** | Multi-target tracking technique handling ambiguous sensor associations. |
| **IMM (Interacting Multiple Models)** | Tracking algorithm that runs multiple motion models in parallel (e.g., constant velocity + constant turn) and weights them. |

## Software & Safety Terms

| Term | One-line definition |
|---|---|
| **AUTOSAR Classic** | Static, statically-configured automotive software architecture for microcontrollers. |
| **AUTOSAR Adaptive** | Newer C++14-based architecture for Linux/QNX ADAS controllers. |
| **ARA::COM** | Adaptive AUTOSAR communication API — service-oriented (proxy/skeleton). |
| **ASIL-D** | Highest ISO 26262 safety integrity level. AEB, brake-by-wire, steer-by-wire are ASIL-D. |
| **MISRA C++** | Coding guidelines for safety-critical C++ (no goto, no implicit conversions, etc.). |
| **HARA** | Hazard Analysis and Risk Assessment — required document for ASIL classification. |
| **Watchdog timer** | Hardware timer that resets the ECU if software hangs. |
| **Diverse redundancy** | Same algorithm implemented two different ways, cross-checked. Required for ASIL-D. |
| **MC/DC coverage** | Modified Condition/Decision Coverage — required test coverage for ASIL-D code. |
| **SOTIF (ISO 21448)** | Safety Of The Intended Functionality — covers hazards from limitations of perception (e.g., camera in fog). |
| **WP.29 / R157** | UN regulations for automated driving features. |
| **OTA (Over-The-Air)** | Software updates pushed to deployed vehicles. |

---

# Part 3 — The Interview Drill (Memorize and Practice Aloud)

When the interviewer asks **"Tell me about yourself,"** the script:

> "I'm a Master's in Computer Science with a focus on systems and software engineering. I've spent the last three days deeply diving into ADAS engineering specifically — building a working end-to-end stack from sensor data analysis through real-time embedded firmware to control theory and sensor fusion. On the perception side I've built sensor log analytics pipelines in pandas/NumPy. On the embedded side I've implemented a real-time AEB demo on ESP32 with sub-2ms control-loop timing. On the algorithm side I've implemented PID with Ziegler-Nichols tuning, linear Kalman Filter for GPS+IMU fusion, and EKF with CTRA motion model for curved-path tracking. I understand the production architecture — AUTOSAR Adaptive on a Linux domain controller, communicating with Classic AUTOSAR microcontrollers for actuator control, all integrated via Automotive Ethernet — and I'm ready to contribute to a Tier-1 or OEM team building production ADAS features."

That's a 60-second answer. Practice it until it's smooth.

---

## The Five Interview Power Questions (Practice Each Aloud)

### Q1: "Walk me through how AEB works."

> "AEB starts with the perception layer. A 77-GHz forward radar provides range and velocity for tracked objects at 50-100 Hz. A forward camera classifies them — pedestrian, cyclist, vehicle. A sensor fusion node combines them into a unified object list with position, velocity, classification confidence. The decision layer computes Time-To-Collision continuously — TTC equals range divided by closing velocity. When TTC drops below threshold (typically 2.6 seconds for highway, 1.5 seconds for urban), and the driver hasn't initiated braking, the system commands full brake pressure via the brake-by-wire actuator, simultaneously sending engine torque cut to the powertrain ECU and pre-tensioning the seatbelts via the body controller. Total latency budget from initial radar detection to brake actuation is 100 milliseconds. ASIL-D rating drives the rigor — diverse redundancy, MC/DC test coverage, MISRA-compliant code, watchdog timers. I implemented a simplified version of this on ESP32 with ultrasonic sensing."

### Q2: "What's the difference between Kalman Filter and Extended Kalman Filter?"

> "Standard Kalman Filter assumes linear state transition and measurement models — multiplications and additions only, no trig or division. Real vehicle motion involves heading-dependent terms like v·cos(θ), which are nonlinear. EKF handles this by computing Jacobians — first-order partial derivatives of the nonlinear functions — at the current state estimate, then applying standard Kalman math to that local linear approximation. EKF works well when nonlinearity is mild over one timestep. For sharp turns or large dt, the linearization error accumulates and the filter can diverge. Mitigations are smaller timesteps, better initialization, or switching to UKF which propagates sigma points through the nonlinear function instead of linearizing."

### Q3: "Why is heap allocation banned in safety-critical real-time loops?"

> "Heap allocation latency is non-deterministic. The C++ allocator must search a free list, possibly split or coalesce blocks, possibly request more memory from the OS. The result is variable execution time — could be 50 nanoseconds, could be 50 microseconds. ISO 26262 ASIL-D requires worst-case execution time analysis — you have to prove your code completes within its budget every time. Variable-time operations make that proof impossible. Production practice is to pre-allocate all buffers at startup and use only stack memory and fixed-size containers in the hot loop. AUTOSAR Adaptive includes a memory pool allocator specifically for bounded-latency dynamic allocation."

### Q4: "How would you debug a production AEB false positive?"

> "I'd start by pulling the log for that drive — radar object track history, camera classification confidence over time, ego vehicle state. Specifically I'd look for ghost radar targets (multipath reflections from overpasses or guard rails), camera misclassifications under unusual lighting, and time-to-collision spikes from miscalculated relative velocity. I'd replay the scenario in simulation — CARLA or IPG CarMaker — to confirm reproducibility. Once identified, mitigations could include radar track confidence thresholding, multi-frame agreement before classifying, or sensor fusion vote-majority. Production teams maintain regression test suites of every reported false positive — new code changes must not re-trigger any known case."

### Q5: "How do you tune Q and R in a Kalman Filter?"

> "Start with R from the sensor datasheet — R is the squared standard deviation of the measurement noise. A GPS spec might say 2.5m at 1-sigma, so R = 6.25. Then estimate Q empirically. Q represents process model error — how much the real dynamics deviate from your assumed model. I tune Q by running the filter on logged data with known ground truth and watching the NIS statistic — Normalized Innovation Squared. If NIS is consistently above the expected chi-square value, the filter is overconfident and Q is too small. If NIS is below, Q is too large. Production teams iterate Q across thousands of recorded drives to converge on values that produce statistically consistent estimates."

---

# Part 4 — The 7-Day Sprint to Interview-Ready

You have OPT time pressure. Here's the exact path.

## Day 4 (Tomorrow) — Production-Tooling Upgrade

Convert one of your existing projects to use a **production library** so you can name-drop it.

**Task:** Reimplement your Day 3 EKF using **Eigen in C++** instead of NumPy in Python.

Why this matters: Eigen is the production sensor-fusion library. Every C++ ADAS interview will probe it. Two hours of work transforms your portfolio.

Resource: https://eigen.tuxfamily.org/dox/group__TutorialMatrixClass.html

Resume bullet after: "Reimplemented EKF in modern C++17 using Eigen for production-grade performance and integration with embedded toolchains."

## Day 5 — ROS2 Integration

**Task:** Set up ROS2 Humble. Create a publisher node that streams your Day 1 sensor data, and a subscriber that runs your Day 3 Kalman Filter on it.

Why this matters: ROS2 is the dominant middleware for robotics and AV. Every Tier-1 development environment has it. Job descriptions list it.

Skip-the-pain version: Use the official `robot_localization` package — feed it your simulated IMU and GPS data, watch it run a production-grade UKF.

Resume bullet: "Integrated sensor fusion pipeline with ROS2 middleware using publisher-subscriber architecture, demonstrating service-oriented design pattern used in Adaptive AUTOSAR."

## Day 6 — CARLA Simulation

**Task:** Install CARLA. Spawn a car. Drive it in autopilot mode. Record a sensor log. Open it in your Day 1 pandas pipeline.

Why this matters: CARLA is the dominant open-source AV simulator. Mentioned in 30%+ of ADAS job descriptions.

Resume bullet: "Built end-to-end perception-to-control simulation in CARLA: data logging, offline analysis, and replay validation."

## Day 7 — Write Two Blog Posts

Not optional. Hiring managers Google candidates. A blog post titled "Building a Production-Grade EKF for ADAS Localization" turns up in those searches. It signals seriousness.

Topics:
1. "From NumPy EKF to Production Eigen: Lessons Learned"
2. "Why AEB Latency Budget Matters: A 1.3ms Demo on ESP32"

Post to Medium or your own static site. Cross-post to LinkedIn.

## Day 8-10 — Interview Practice

Record yourself answering the 5 power questions. Listen back. Cringe. Re-record. Listen. Cringe less. Repeat until each answer is under 60 seconds, fluent, and natural.

Schedule mock interviews on Pramp or Interviewing.io. Free.

## Day 11+ — Apply Aggressively

Target list (in approachability order):
1. **Aptiv** (Boston, Pittsburgh) — strong Tier-1, sponsors H-1B
2. **Continental** (Auburn Hills, Silicon Valley) — many entry positions
3. **Bosch** (Plymouth MI, Sunnyvale) — high sponsorship volume
4. **ZF, Magna, Valeo** — mid-tier Tier-1s, less competition
5. **Plus, Gatik, Embark** — trucking-focused AV startups, OPT-friendly
6. **Aurora, Motional** — AV companies, competitive but real openings

Search terms (you already have these):
- "ADAS Software Engineer"
- "ADAS Validation Engineer"
- "Sensor Fusion Engineer"
- "Perception Software Engineer"
- "Robotics Software Engineer — Autonomy"

---

# Part 5 — Resume Section You Can Copy Today

**Projects**

**ADAS Localization & Control Pipeline** *(GitHub: adas-day03)*
Built complete sensor fusion and control stack including discrete PID with Ziegler-Nichols tuning for ACC speed regulation, linear Kalman Filter fusing GPS + IMU achieving 5x position-error improvement over IMU dead-reckoning, and Extended Kalman Filter with CTRA motion model and Jacobian-based linearization for curved trajectory tracking. Demonstrates production localization architecture used in Tier-1 ADAS systems.

**Real-Time AEB Prototype on ESP32** *(GitHub: adas-day02)*
Implemented Automatic Emergency Braking demo on simulated ESP32 hardware with ultrasonic sensing, fixed-size ring-buffer noise filtering, and threshold-based emergency triggering. Benchmarked control loop at 1.3ms cycle time against 10ms ASIL-D real-time budget. Demonstrates pre-allocated-buffer pattern and deterministic execution required by ISO 26262.

**Vehicle Sensor Log Analytics** *(GitHub: adas-sensor-analytics)*
Built pandas/NumPy pipeline for vehicle telemetry analysis with boolean-masking event detection (hard-braking, sharp-turn) and rolling-window noise filtering. Demonstrated 100x speedup via NumPy vectorization vs Python loops. Designed to scale toward Parquet/Spark for fleet-level deployment.

**Skills (add to your skills section)**
- Languages: C++17, C, Python (pandas, NumPy, matplotlib, scipy)
- ADAS: Kalman Filter, Extended Kalman Filter, PID control, sensor fusion, CTRA motion model
- Embedded: ESP32, real-time control loops, fixed-buffer patterns, std::chrono timing
- Tools: ROS2 (basics), Wokwi, CARLA (familiar), Git, GitHub Actions, Jupyter
- Standards: ISO 26262 (concepts), MISRA C++ (concepts), AUTOSAR (concepts), ASIL framework

---

# Part 6 — What Production Code Actually Looks Like (Quick Glance)

Just so you've seen it. Here's what your Day 3 EKF looks like in production C++ using Eigen:

```cpp
#include <Eigen/Dense>

class CTRAExtendedKalmanFilter {
public:
    using State = Eigen::Matrix<double, 5, 1>;          // [x, y, v, theta, omega]
    using StateCovariance = Eigen::Matrix<double, 5, 5>;
    using Measurement = Eigen::Matrix<double, 2, 1>;    // GPS x, y
    using MeasurementCovariance = Eigen::Matrix<double, 2, 2>;

    void Predict(double dt, double imu_yaw_rate) {
        const double x     = state_(0);
        const double y     = state_(1);
        const double v     = state_(2);
        const double theta = state_(3);
        
        // Nonlinear state propagation
        State predicted;
        predicted << x + v * std::cos(theta) * dt,
                     y + v * std::sin(theta) * dt,
                     v,
                     theta + imu_yaw_rate * dt,
                     imu_yaw_rate;
        
        // Jacobian
        Eigen::Matrix<double, 5, 5> F = Eigen::Matrix<double, 5, 5>::Identity();
        F(0, 2) =  std::cos(theta) * dt;
        F(0, 3) = -v * std::sin(theta) * dt;
        F(1, 2) =  std::sin(theta) * dt;
        F(1, 3) =  v * std::cos(theta) * dt;
        F(3, 4) =  dt;
        
        state_      = predicted;
        covariance_ = F * covariance_ * F.transpose() + process_noise_;
    }

    void Update(const Measurement& gps_measurement) {
        const Measurement innovation = gps_measurement - measurement_matrix_ * state_;
        const MeasurementCovariance innovation_cov =
            measurement_matrix_ * covariance_ * measurement_matrix_.transpose() + measurement_noise_;
        
        const Eigen::Matrix<double, 5, 2> kalman_gain =
            covariance_ * measurement_matrix_.transpose() * innovation_cov.inverse();
        
        state_      += kalman_gain * innovation;
        covariance_ -= kalman_gain * measurement_matrix_ * covariance_;
    }

private:
    State state_ = State::Zero();
    StateCovariance covariance_ = StateCovariance::Identity() * 10.0;
    StateCovariance process_noise_;
    MeasurementCovariance measurement_noise_;
    Eigen::Matrix<double, 2, 5> measurement_matrix_;
};
```

That's production-quality. Same algorithm you built. Modern C++17, Eigen, type-safe templates, RAII destructors implicit, no raw pointers, no manual memory management. **You already understand every line.** You just don't write it that way yet. Day 4 task fixes that.

---

# Final Reality Check

You're not behind. You're three days into ADAS and you've built more than 80% of CS grads will ever attempt. The gap to interview-ready is the **production language** and **business framing** of what you already built. That's reframing — not relearning.

What I want you to do **right now, before sleep**:

1. Pick one of the three projects (probably Project 2 — the AEB demo is most concrete).
2. Practice the production framing paragraph **out loud, three times**.
3. Practice the business case **out loud, three times**.
4. Tomorrow morning's first task: convert your Day 3 EKF to C++ Eigen (I'll help if you want — it's one 90-minute session).

You're not learning anymore. You're **packaging and selling** what you already know.

— **End of Production Reality Guide**
