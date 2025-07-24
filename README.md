# MeNow Smell Prediction

**Multi-label odor classification using ChemBERTa and traditional ML approaches**

Predicting molecular odor properties (sweet, floral, minty, pungent) from SMILES strings using state-of-the-art transformer models and classical machine learning methods.

---

## 🎯 **Overview**

This project implements a complete pipeline for predicting multiple odor labels from molecular structures. We use two complementary approaches:

1. **🧬 ChemBERTa + LoRA**: Parameter-efficient fine-tuning of a chemical language model
2. **⚗️ Traditional ML**: LightGBM/XGBoost with molecular fingerprints and descriptors

Both approaches handle the multi-label nature of odor prediction, where molecules can exhibit multiple odor characteristics simultaneously.

---

## 📊 **Dataset**

- **Source**: Curated GoodScents dataset from pyrfume-data repository
- **Training**: 1,460 molecules
- **Test**: 366 molecules  
- **Labels**: 4 odor classes (sweet, floral, minty, pungent)
- **Format**: Multi-label classification (molecules can have 0-4 labels)

### Label Distribution:
- **Sweet**: 55.2% of molecules
- **Floral**: 49.7% of molecules  
- **Minty**: 10.6% of molecules
- **Pungent**: 6.1% of molecules

---

## 🚀 **Quick Start**

### 1. Setup Environment
```bash
conda create -n scent-env python=3.10
conda activate scent-env
pip install transformers torch datasets peft accelerate
pip install lightgbm xgboost scikit-learn
pip install rdkit matplotlib seaborn pandas numpy
```

### 2. Run ChemBERTa Fine-tuning
```bash
# Local training (20 epochs)
python chemberta_odor_finetuning.py --epochs 20 --seed 42

# Quick test (2 epochs)  
python chemberta_odor_finetuning.py --epochs 2 --batch_size 8 --seed 42

# HuggingFace/Colab compatible
python run_on_huggingface.py
```

### 3. Run Traditional ML Pipeline
```bash
# Both LightGBM and XGBoost
python ml_odor_classification.py --models lightgbm xgboost --seed 42

# With GPU acceleration (HuggingFace)
python ml_odor_classification.py --models lightgbm xgboost --gpu --seed 42

# Custom molecular features
python ml_odor_classification.py --morgan_bits 4096 --seed 42
```

---

## 🏗️ **Repository Structure**

```
MeNow-smell_pred/
├── chemberta_odor_finetuning.py    # ChemBERTa LoRA fine-tuning
├── chemberta_odor_predict.py       # Inference with trained ChemBERTa model
├── ml_odor_classification.py       # Traditional ML pipeline
├── run_on_huggingface.py          # HuggingFace/Colab runner
├── explore_bushdid_chemberta.ipynb # Data exploration notebook
├── goodscents_train.csv           # Training data
├── goodscents_test.csv            # Test data  
├── bushdid_predict.csv            # Prediction targets
├── pyrfume-data/                  # Raw datasets from pyrfume
└── results/                       # Model outputs and plots
    ├── chemberta_lora_results/    # ChemBERTa results
    └── ml_odor_results/           # Traditional ML results
```

---

## 🧬 **ChemBERTa Approach**

### Model Architecture
- **Base Model**: `seyonec/SMILES_tokenized_PubChem_shard00_160k` (ChemBERTa)
- **Fine-tuning**: LoRA (Low-Rank Adaptation) for efficiency
- **Parameters**: Only 1.05% of model parameters trained (888K vs 83M)
- **Output**: 4 sigmoid neurons for multi-label classification
- **Loss**: Binary Cross-Entropy with class weights for imbalance

### Key Features
- ✅ **Memory Efficient**: LoRA reduces memory usage by ~90%
- ✅ **Multi-Platform**: CUDA, Apple MPS, CPU compatible
- ✅ **Class Weights**: Handles imbalanced labels (minty: 8.4x, pungent: 15.4x)
- ✅ **Reproducible**: Fixed seeds for consistent results
- ✅ **Early Stopping**: Prevents overfitting
- ✅ **Comprehensive Metrics**: ROC-AUC, PR-AUC, F1 scores per class

### Training Configuration
```python
NUM_EPOCHS = 20
BATCH_SIZE = 16  
LEARNING_RATE = 5e-4
LORA_R = 16
WARMUP_STEPS = 100
```

---

## ⚗️ **Traditional ML Approach**

### Feature Engineering
- **Morgan Fingerprints**: 2048-bit molecular fingerprints
- **RDKit Descriptors**: Molecular weight, LogP, H-bond donors/acceptors, TPSA, etc.
- **Feature Count**: 2048 + 10 = 2058 features total

### Models
- **LightGBM**: Gradient boosting with GPU support
- **XGBoost**: Gradient boosting with CUDA support  
- **Multi-Output**: Separate binary classifier for each odor label

### Key Features
- ✅ **GPU Acceleration**: Both LightGBM and XGBoost support GPU
- ✅ **Class Weights**: Handles imbalanced data
- ✅ **Fast Training**: Much faster than deep learning approaches
- ✅ **Interpretable**: Feature importance analysis
- ✅ **Multi-Label Analysis**: Label co-occurrence and correlation analysis

---

## 📈 **Results & Performance**

### ChemBERTa LoRA Results (2 epochs test run)
| Metric | Score |
|--------|-------|
| **Macro F1** | 0.6068 |
| **Micro F1** | 0.8060 |
| **Exact Match** | 0.3934 |
| **Hamming Loss** | 0.1940 |

### Per-Class Performance
| Class | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|-------|-----------|--------|----|---------| -------|
| **Sweet** | 0.649 | 0.728 | 0.687 | 0.665 | 0.737 |
| **Floral** | 0.672 | 0.902 | 0.770 | 0.805 | 0.778 |
| **Minty** | 0.700 | 0.600 | 0.646 | 0.902 | 0.612 |
| **Pungent** | 0.375 | 0.286 | 0.324 | 0.870 | 0.391 |

### Training Efficiency
- **Training Time**: 2.68 minutes (2 epochs on MacBook Air M2)
- **Memory Usage**: ~90% less than full fine-tuning
- **Model Size**: ~3MB LoRA adapters vs ~300MB full model

---

## 🎨 **Visualizations**

Both approaches generate comprehensive plots:

### ChemBERTa Plots
- **ROC Curves**: Per-class ROC-AUC analysis
- **Precision-Recall Curves**: PR-AUC for imbalanced classes
- **Training History**: Loss curves, F1 scores, learning rate schedule
- **Performance Summary**: Class distribution and metrics comparison

### Traditional ML Plots  
- **Model Comparison**: F1 scores across LightGBM vs XGBoost
- **Per-Class Performance**: ROC-AUC and F1 by odor class
- **Multi-Label Analysis**: Label co-occurrence, correlation matrix
- **Error Distribution**: Hamming distance and prediction complexity

---

## 🔬 **Multi-Label Handling**

Both approaches properly handle the multi-label nature of odor prediction:

### ChemBERTa Approach
- **Independent Predictions**: Each label predicted independently
- **Sigmoid Outputs**: Can predict multiple labels per molecule
- **BCE Loss**: Binary cross-entropy for each label
- **Class Weights**: Address label imbalance (pungent: 15.4x weight)

### Traditional ML Approach
- **MultiOutputClassifier**: Separate binary classifier per label
- **Label Analysis**: Co-occurrence patterns and correlations
- **Imbalance Handling**: Class weights and evaluation metrics
- **Pattern Detection**: Most common label combinations

---

## 🔧 **Advanced Usage**

### Custom ChemBERTa Training
```bash
# Extended training with custom parameters
python chemberta_odor_finetuning.py \
    --epochs 50 \
    --batch_size 32 \
    --learning_rate 3e-4 \
    --lora_r 32 \
    --seed 42
```

### Custom ML Pipeline
```bash
# Custom feature engineering
python ml_odor_classification.py \
    --models lightgbm \
    --morgan_bits 4096 \
    --no_features \
    --seed 42
```

### Prediction on New Molecules
```bash
# Use trained ChemBERTa model
python chemberta_odor_predict.py \
    --model_path ./chemberta_lora_results/final_model \
    --input new_molecules.csv \
    --output predictions.csv
```

---

## 📊 **Evaluation Metrics**

### Multi-Label Metrics
- **Exact Match** (Subset Accuracy): All labels correct
- **Hamming Loss**: Average per-label error rate  
- **Jaccard Score**: Label set overlap
- **Micro F1**: Treats each label prediction independently
- **Macro F1**: Average across label classes

### Per-Class Metrics
- **ROC-AUC**: Area under ROC curve
- **PR-AUC**: Area under Precision-Recall curve (better for imbalanced data)
- **Precision/Recall/F1**: Standard classification metrics

---

## 💡 **Key Insights**

### Model Performance
- **ChemBERTa excels** at capturing complex molecular patterns
- **Traditional ML** provides faster training and interpretability
- **Multi-label prediction** is challenging due to label imbalance
- **Class weights** are crucial for minority classes (minty, pungent)

### Label Characteristics
- **Sweet & Floral**: Easier to predict (balanced, high frequency)
- **Minty & Pungent**: Harder due to low frequency and imbalance
- **Label Correlations**: Some odors co-occur more frequently
- **Molecular Complexity**: More labels per molecule increase difficulty

---

## 🔄 **Reproducibility**

### Fixed Seeds
- Python random: 42
- NumPy: 42  
- PyTorch: 42
- Model training: 42

### Environment
```bash
# Complete environment setup
conda env create -f environment.yml
conda activate scent-env
```

### Data Splits
- Consistent train/test splits stored in CSV files
- All preprocessing steps documented and reproducible

---

## 📝 **Future Improvements**

### Model Enhancements
- [ ] Ensemble ChemBERTa + Traditional ML predictions
- [ ] Attention visualization for model interpretability
      
### Data Augmentation
- [ ] SMILES augmentation (randomized notation)
- [ ] Molecular scaffolds and fragments
- [ ] Semi-supervised learning with unlabeled molecules
- [ ] Additional odor classes and datasets

### Deployment
- [ ] REST API for real-time predictions
- [ ] Integration with molecular design tools

---

## 📚 **References**

- **ChemBERTa**: [Chithrananda et al., 2020](https://arxiv.org/abs/2010.09885)
- **LoRA**: [Hu et al., 2021](https://arxiv.org/abs/2106.09685) 
- **Pyrfume**: [Sanchez-Lengeling et al., 2019](https://arxiv.org/abs/1910.10685)
- **RDKit**: [Landrum, G. (2006)](https://www.rdkit.org/)

---

## 📄 **License**

MIT License - see [LICENSE](LICENSE) file for details.

---



