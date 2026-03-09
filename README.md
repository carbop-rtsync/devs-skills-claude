# DEVS Skills for Claude Code

A collection of Claude Code skills for building **Discrete Event System Specification (DEVS)** and **PowerDEVS** simulation models through AI-assisted, structured workflows.

These skills are designed for use with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and the [claude-flow](https://github.com/ruvnet/claude-flow) agent orchestration framework.

---

## Skills Included

### power-devs-builder-startup

**General-purpose PowerDEVS model builder for continuous and hybrid system simulation.**

Creates complete PowerDEVS models (`.pdm` files) using Quantized State Systems (QSS) methods. Given a brief system description, the skill autonomously:

1. Scans a reference library of 306 atomic block headers and 44 model templates
2. Infers the complete model structure (integrators, summers, multipliers, signal routing)
3. Generates a valid `.pdm` file with correct parameters and connections
4. Validates all atomic paths and parameters against the PowerDEVS installation
5. Deploys the model to the PowerDEVS simulator
6. Produces a summary document

**Use for:** Differential equations, control systems, population dynamics, electrical circuits, hybrid systems, or any continuous-time system modeled with PowerDEVS.

**Runtime prompts:** This is the distributable version. It prompts for user-specific directories (PowerDEVS install path, project directory) at runtime rather than using hardcoded paths.

#### Workflow

```
Phase 0: SCAN       Catalog reference library (306 atomics, 44 models)
Phase 1: DISCOVER   Infer equations, components, routing from description
Phase 2: GENERATE   Build .pdm file (atomics, junctions, connections)
Phase 3: VALIDATE   Verify paths, parameters, GnuPlot config; auto-correct
Phase 4: DEPLOY     Copy to PowerDEVS WSL installation
Phase 5: SUMMARIZE  Generate docs/models/[name].md
```

#### Reference Library

The skill includes a self-contained reference library under `resources/`:

| Category | Contents | Count |
|----------|----------|-------|
| **Atomic Blocks** | QSS integrators, multipliers, saturation, sources, sinks, hybrid blocks | 306 headers |
| **Reference Models** | Lotka-Volterra, Van der Pol, buck converters, bouncing ball, Petri nets, networks | 44 `.pdm` files |
| **Templates** | Reusable model starting points | varies |

<details>
<summary>Atomic block categories (21 directories)</summary>

| Directory | Files | Description |
|-----------|-------|-------------|
| `qss/` | 48 | QSS1-4, LIQSS, BQSS, CQSS integrators, weighted sums, multipliers, trig/exp functions |
| `hybrid/` | 44 | Hybrid continuous/discrete blocks: integrators, switches, saturation, buck converters, neurons |
| `continuous/` | 33 | Classical continuous blocks: integrators, multipliers, gain, delay, math expressions |
| `sources/` | 32 | Signal generators: step, ramp, pulse, sine, square, random, constant, file input, custom steps |
| `sinks/` | 28 | Output blocks: GnuPlot, data logger, file output, Scilab interface, oscilloscope |
| `vector/` | 28 | Large-scale vectorized operations: vector integrators, sums, gains, coupling |
| `network/` | 38 | Data network simulation: routers, queues, servers, links, traffic generators |
| `hybrid_network/` | 8 | Hybrid network blocks |
| `modelica/` | 8 | Modelica interface blocks |
| `petri/` | 6 | Petri net components |
| `adquisicion/` | 6 | Data acquisition blocks |
| `queueing/` | 5 | Queueing theory blocks |
| `experimental/` | 5 | Experimental features |
| `realtime/` | 5 | Real-time execution blocks |
| `control/` | 3 | Control system blocks |
| `discrete/` | 2 | Discrete event blocks |
| `signal/` | 2 | Signal processing |
| `udpcomm/` | 2 | UDP communication |
| `random/` | 1 | Random number generation |
| `general/` | 1 | General utilities |
| `signal_bus/` | 1 | Signal bus routing |

</details>

<details>
<summary>Reference models by domain</summary>

| Domain | Models |
|--------|--------|
| **Continuous Systems** | Lotka-Volterra, Van der Pol oscillator, nonlinear stiff, LC transmission line |
| **Hybrid Systems** | Buck converters (controlled, coupled, discrete), DC motor drives, bouncing ball, spiking neurons, inverted pendulum |
| **Delay Differential Equations** | Cellular network spikes, Hairer benchmark, Oberle-Pesch problem |
| **Discrete Event** | Nicholson-Bailey host-parasitoid, Q-operator, state-space |
| **Networks** | Basic topology, advanced networks (2 variants) |
| **Petri Nets** | Dining philosophers, queue processor |
| **Queueing** | Basic queueing model |
| **Vector/Large-Scale** | AIRS satellite, inverter arrays, neural networks |
| **Test/Validation** | QSS validation, bounded integrator, comparator, reservoir, buffer server, saturation |

</details>

---

## Prerequisites

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** installed and configured
- **[PowerDEVS](https://sourceforge.net/projects/powerdevs/)** simulator (for running generated models)
  - Typically installed under WSL on Windows systems
- **WSL** (Windows Subsystem for Linux) if running on Windows

---

## Installation

### Option 1: Clone and register as a Claude Code skill

```bash
# Clone this repository
git clone https://github.com/carbop-rtsync/devs-skills-claude.git

# Copy the skill into your project's .claude/skills/ directory
cp -r devs-skills-claude/power-devs-builder-startup /path/to/your/project/.claude/skills/
```

### Option 2: Add directly to an existing project

```bash
# From your project root
mkdir -p .claude/skills
cd .claude/skills
git clone https://github.com/carbop-rtsync/devs-skills-claude.git
# Move skill to expected location
mv devs-skills-claude/power-devs-builder-startup ./
```

### Option 3: Use with claude-flow

If using the [claude-flow](https://github.com/ruvnet/claude-flow) orchestration framework, skills in `.claude/skills/` are automatically discovered:

```bash
# Install claude-flow
npm install -g @claude-flow/cli@latest

# Initialize in your project
npx @claude-flow/cli@latest init

# Copy skill into the project's skill directory
cp -r devs-skills-claude/power-devs-builder-startup .claude/skills/
```

### Verify installation

Once installed, the skill appears in Claude Code's skill list. Invoke it by name:

```
/power-devs-builder-startup
```

Or describe your system and Claude Code will match the skill automatically when relevant.

---

## Usage

### Quick Start

1. Open Claude Code in your project directory
2. Describe the system you want to model:

   > "Model a predator-prey system with Lotka-Volterra dynamics: dx/dt = ax - bxy, dy/dt = cxy - dy"

3. The skill walks through 6 phases automatically:
   - Scans the reference library
   - Analyzes your description and infers equations, components, routing
   - Generates the `.pdm` file
   - Validates against your PowerDEVS installation
   - Deploys to the simulator
   - Produces documentation

4. On first run, you will be prompted for:
   - **PowerDEVS WSL path** (e.g., `~/powerdevs-CERN`)
   - **Project working directory** (e.g., `C:\Users\yourname\Projects\MyModel`)

### Example Systems

| System | Description |
|--------|-------------|
| Lotka-Volterra | Predator-prey population dynamics with 2 ODEs |
| Mass-Spring-Damper | Mechanical oscillator: dx1/dt = x2, dx2/dt = (F - cx2 - kx1)/m |
| Buck Converter | Switched-mode power supply with hybrid dynamics |
| Inventory Model | IRP retailer inventory with pulsed deliveries and continuous consumption |
| Lorenz Attractor | 3-ODE chaotic system: dx/dt = sigma(y-x), dy/dt = x(rho-z)-y, dz/dt = xy - beta*z |

---

## Skill Architecture

### SKILL.md

Each skill is defined by a `SKILL.md` file with YAML frontmatter:

```yaml
---
name: "PowerDEVS Builder Startup"
description: "Create PowerDEVS models (.pdm files) for continuous/hybrid system simulation..."
---
```

The body contains the full workflow specification including:
- Phase definitions with concurrent execution diagrams
- MODEL_CONTEXT shared state object
- Component catalog and parameter references
- Validation rules and auto-correction logic
- Deployment and summary generation

### Resources Directory

```
power-devs-builder-startup/
├── SKILL.md              # Skill definition (1,319 lines)
└── resources/
    ├── README.md         # Resource catalog documentation
    ├── atomics/          # 306 atomic block headers (.h)
    │   ├── qss/          # 48 - QSS integration methods
    │   ├── continuous/   # 33 - Continuous system blocks
    │   ├── hybrid/       # 44 - Hybrid system blocks
    │   ├── sinks/        # 28 - Output/visualization
    │   ├── sources/      # 32 - Signal generators
    │   └── ...           # 16 more categories
    ├── models/           # 44 reference .pdm model files
    └── templates/        # Reusable model templates
```

### MODEL_CONTEXT

A shared state object persists across all 6 phases, maintaining semantic continuity:

```
MODEL_CONTEXT = {
  user_config:      // Runtime directory paths (collected once)
  catalog:          // Reference library index (Phase 0)
  model:            // Name, description, goal (Phase 1)
  equations:        // Differential and algebraic equations (Phase 1)
  state_variables:  // State vars with initial values (Phase 1)
  parameters:       // Model parameters (Phase 1)
  qss_config:       // QSS method, quantum settings (Phase 1)
  components:       // Atomic blocks with positions (Phase 2)
  connections:      // Signal routing lines (Phase 2)
  junctions:        // Junction points (Phase 2)
  validation:       // Path, parameter, GnuPlot checks (Phase 3)
  deployment:       // WSL deployment status (Phase 4)
  summary:          // Documentation path (Phase 5)
}
```

---

## Related Skills & Resources

### Companion Skills (not included in this repo)

These skills are available separately and complement the PowerDEVS builder:

| Skill | Source | Description |
|-------|--------|-------------|
| **devs-builder** | Local / [claude-flow](https://github.com/ruvnet/claude-flow) | Builds discrete-event DEVS models in Java for the MS4Systems framework. Generates `.java` atomic and coupled models with test harnesses. |
| **my-first-skill** | Local | Creates a simple DEVS model outline using Basic DEVS Formalism. Introductory skill for learning DEVS concepts. |
| **power-devs-builder** | Local | Personalized version of this skill with hardcoded directory paths. Not for distribution. |

### claude-flow Agent Orchestration

[claude-flow](https://github.com/ruvnet/claude-flow) (by [ruvnet](https://github.com/ruvnet)) is the enterprise-grade AI agent orchestration framework that powers skill execution in Claude Code. It provides:

- **60+ specialized agent types** (coder, tester, reviewer, architect, security auditor, etc.)
- **Multi-agent swarm coordination** with hierarchical, mesh, and adaptive topologies
- **42+ built-in skills** across development, testing, security, GitHub, and SPARC methodology
- **Self-learning hooks** that adapt routing based on successful patterns
- **AgentDB** with HNSW vector search for persistent agent memory
- **MCP integration** for native Claude Code tool access

Skills from this repository can be used standalone or combined with claude-flow's orchestration for more complex workflows.

#### Installing claude-flow

```bash
# Quick setup
npm install -g @claude-flow/cli@latest

# Or via npx (no install)
npx @claude-flow/cli@latest init

# Add MCP integration to Claude Code
claude mcp add claude-flow -- npx -y @claude-flow/cli@latest
```

See the [claude-flow documentation](https://github.com/ruvnet/claude-flow) for full setup and usage.

### PowerDEVS Simulator

[PowerDEVS](https://sourceforge.net/projects/powerdevs/) is a general-purpose software tool for DEVS modeling and simulation of hybrid systems using QSS (Quantized State Systems) methods. It was developed at the University of Buenos Aires and CIFASIS-CONICET.

Key references:
- Bergero, F. and Kofman, E. (2011). PowerDEVS: a tool for hybrid system modeling and real-time simulation. *Simulation*, 87(1-2), 113-132.
- Kofman, E. (2004). Discrete Event Simulation of Hybrid Systems. *SIAM Journal on Scientific Computing*, 25(5), 1771-1797.

### DEVS Formalism

DEVS (Discrete Event System Specification) is a modular, hierarchical formalism for modeling and analyzing systems:

- **Atomic DEVS**: M = <X, Y, S, delta_int, delta_ext, lambda, ta>
- **Coupled DEVS**: N = <X, Y, D, {M_d}, {I_d}, {Z_ij}, Select>

Key reference:
- Zeigler, B.P., Praehofer, H., and Kim, T.G. (2000). *Theory of Modeling and Simulation*. Academic Press.

---

## Contributing

To add new skills to this repository:

1. Create a new directory under the repo root with your skill name
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`)
3. Include a `resources/` directory with any reference files the skill needs
4. Ensure all paths are generic (no hardcoded user directories)
5. Submit a pull request

### Skill structure

```
your-skill-name/
├── SKILL.md          # Skill definition with YAML frontmatter
└── resources/        # Reference files (optional)
    ├── models/       # Reference models
    ├── templates/    # Reusable templates
    └── ...           # Domain-specific resources
```

### SKILL.md format

```yaml
---
name: "Your Skill Name"
description: "One-line description of what the skill does."
---

# Your Skill Name

## What This Skill Does
[Description of the skill's purpose and workflow]

## Workflow Overview
[Phase-by-phase breakdown]

...
```

---

## License

This repository contains skills and reference materials for academic and research use in DEVS and PowerDEVS modeling and simulation.

PowerDEVS atomic block headers are sourced from the [PowerDEVS](https://sourceforge.net/projects/powerdevs/) project (CIFASIS-CONICET / Universidad Nacional de Rosario).

---

## Acknowledgments

- **[claude-flow](https://github.com/ruvnet/claude-flow)** by [ruvnet](https://github.com/ruvnet) -- Agent orchestration framework and skills infrastructure
- **[PowerDEVS](https://sourceforge.net/projects/powerdevs/)** by Federico Bergero, Ernesto Kofman et al. -- QSS-based hybrid system simulator
- **[MS4Systems](https://ms4systems.com/)** -- DEVS modeling and simulation framework for Java
- **SIW 2026 DEVS Tutorial** -- Inventory Routing Problem case study (simlytics-cloud/irp-adevs)
