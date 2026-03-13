# connect4-bot

Neural Network Bot for Connect 4 — CNN & Vision Transformer (ViT)  
RM 294 – Optimization II | MSBA, University of Texas | Fall 2025

**Team:** Group 14

🎮 **Live App:** [msba25optim2-14.anvil.app](https://msba25optim2-14.anvil.app/)  
🔗 **Anvil Clone:** [Clone Link](https://anvil.works/build#clone:D6ITCFFII236YTYZ=WDEPADLW7ZZADB6HJMTS245W)  
📦 **Full Submission Repo:** [github.com/manoranjith-a/connect4-bot](https://github.com/manoranjith-a/connect4-bot)

---

## Overview

This project builds a full-stack Connect 4 AI bot by:

1. **Generating training data** using Monte Carlo Tree Search (MCTS)
2. **Training two neural networks** — a CNN and a Vision Transformer (ViT) — to predict the best move for any board position
3. **Deploying the models** via a Dockerized Flask backend on AWS EC2
4. **Building an interactive web app** on Anvil where users can play against the bots

---

## Repository Structure

```
connect4_submission/
│
├── README.txt
│
├── training_code/
│   ├── connect4_cnn_vit.ipynb     # Training notebook (CNN + ViT)
│   ├── inference.py               # Loads .pt weights and predicts moves
│   ├── test_inference.py          # Quick inference checks
│   ├── app/
│   │   ├── cnn_model.py           # CNN architecture (Connect4StrongCNN)
│   │   └── vit_model.py           # Transformer architecture (BoardViTStrong)
│   └── weights/
│       ├── best_cnn_2ch.pt        # Trained CNN weights (PyTorch)
│       └── best_vit_2ch.pt        # Trained ViT weights (PyTorch)
│
└── aws_deployment/
    ├── Dockerfile
    ├── docker-compose.yaml
    ├── backend.py                 # Flask API backend for deployment
    ├── requirements.txt
    └── aws_running_screenshot.png # Proof of EC2 + Docker running
```

---

## Data Generation

Training data was generated using **Monte Carlo Tree Search (MCTS)** self-play.

### Files
| File | Description |
|------|-------------|
| `data_generation.ipynb` | MCTS self-play to generate board-move pairs |
| `dataset_train_small.npz` | Training dataset (board positions + best moves) |
| `dataset_val_small.npz` | Validation dataset |

### How Data Was Generated

1. MCTS plays against itself across many games
2. Occasionally injects **random moves** at the start of games to increase board diversity
3. Records **(board state, recommended move)** pairs — only MCTS-recommended moves are saved, not the random warm-up moves
4. Games are terminated as soon as a winner is found — no post-win moves added
5. Board encoded as **2-channel 6×7 grid** (current player vs opponent)

```
Board encoding (2-channel):
  channel 0: 1 where current player's pieces are, 0 elsewhere
  channel 1: 1 where opponent's pieces are, 0 elsewhere
```

> If the same board appears with different MCTS recommendations across games (due to MCTS randomness), the **most frequently recommended move** is kept.

---

## Models

### CNN — `Connect4StrongCNN`
- Multiple convolutional layers with varying filter sizes
- Max pooling + dropout for regularization
- Dense layers for final move classification (7 output classes = 7 columns)

### Vision Transformer — `BoardViTStrong`
- Board treated as a sequence of patches (image classification framing)
- Multi-head self-attention (MHSA) layers
- Overlapping patch windows for richer spatial context

### Training
- Framework: **PyTorch**
- Input: 2-channel 6×7 board encoding
- Output: 7-class softmax (one per column)
- Illegal move masking applied at inference time
- Trained on Google Colab (GPU)

---

## Deployment

### Architecture
```
User (Browser)
      ↓
Anvil Web App
      ↓
Flask API (AWS EC2 + Docker)
      ↓
PyTorch Model (CNN or ViT)
```

### Run Locally

```bash
cd aws_deployment
docker-compose up --build
```

### AWS EC2
- Dockerized Flask backend hosted on AWS EC2
- Anvil frontend sends board state → backend returns best move
- Screenshot of running instance included in `aws_deployment/`

---

## Web App (Anvil)

🎮 **[msba25optim2-14.anvil.app](https://msba25optim2-14.anvil.app/)**

### Features
- 🔐 Login required (email + password)
- 📖 Detailed write-up: architecture decisions, training process, board examples the model classifies well vs. poorly
- 🎮 Interactive Connect 4 — play against CNN or ViT
- 🔄 New Game button to reset

> **Login credentials for grader:**  
> Email: `dan` | Password: `Optimization1234`

---

## Tech Stack

| Component | Tool |
|-----------|------|
| Data generation | Monte Carlo Tree Search (MCTS) |
| Model training | PyTorch, Google Colab (GPU) |
| CNN | Custom `Connect4StrongCNN` |
| Transformer | Custom `BoardViTStrong` (ViT) |
| Backend API | Flask |
| Containerization | Docker + docker-compose |
| Cloud hosting | AWS EC2 |
| Frontend | Anvil |

---

## License

Completed as coursework for RM 294 – Optimization II at the University of Texas MSBA program. For academic use only.
