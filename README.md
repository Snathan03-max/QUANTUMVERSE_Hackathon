# Quantum Radar for Battleship - QuantumVerse 4.0 Hackathon

This repository contains our implementation for the QuantumVerse 4.0 Hackathon, tackling both Track 1 (Interaction-Free Bomb Tester) and Track 2 (Quantum Radar for Battleship). 

The main idea of this project is based on the Elitzur-Vaidman bomb testing experiment from 1993. In classical physics, if you want to know if an object or bomb is in a room, you have to bounce something off it (like light or a probe). But if that bomb is super sensitive, probing it will trigger it to explode. Quantum mechanics gives us a workaround using superposition and destructive interference. By setting up a Mach-Zehnder interferometer, we can detect whether a target is present without actually transferring energy to it a good fraction of the time.

In this hackathon, we built the core single-cell bomb tester circuit, improved its score, and then scaled that exact principle up to build a 2D Quantum Radar that plays Battleship. Instead of clicking every square one by one like a classical player (which blows up every ship it finds), our radar scans rows and columns in superposition, finding the hidden ships safely with fewer measurements.

---

Interactive Simulation Link
You can try the interactive bomb tester visualizer and simulation demo here:

https://lab.quantumflytrap.com/u/swamin058/AU2MwSMlTF-V1ALcLMmbUg?mode=beam