---
title: "Keras Losses"
description: "Keras to DL4J loss function mapping for model import"
---

## Keras Loss Functions

DL4J supports all standard Keras loss functions except `logcosh`. Loss function mapping is implemented in [KerasLossUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasLossUtils.java).

---

## Loss Mapping Table

| Keras Loss | DL4J Loss Class | Supported |
|---|---|---|
| `mean_squared_error` / `mse` | `LossMSE` | Yes |
| `mean_absolute_error` / `mae` | `LossMAE` | Yes |
| `mean_absolute_percentage_error` / `mape` | `LossMAPE` | Yes |
| `mean_squared_logarithmic_error` / `msle` | `LossMSLE` | Yes |
| `squared_hinge` | `LossSquaredHinge` | Yes |
| `hinge` | `LossHinge` | Yes |
| `categorical_hinge` | `LossCategoricalHinge` | Yes |
| `logcosh` | — | No |
| `categorical_crossentropy` | `LossMCXENT` | Yes |
| `sparse_categorical_crossentropy` | `LossSparseMCXENT` | Yes |
| `binary_crossentropy` | `LossBinaryXENT` | Yes |
| `kullback_leibler_divergence` / `kld` | `LossKLD` | Yes |
| `poisson` | `LossPoisson` | Yes |
| `cosine_proximity` | `LossCosineProximity` | Yes |

---

## Loss Descriptions

### mean_squared_error

Mean of squared differences between predictions and targets. Standard regression loss.

```
MSE = mean((y_true - y_pred)^2)
```

Use for regression tasks with normally distributed errors.

### mean_absolute_error

Mean of absolute differences. More robust to outliers than MSE.

```
MAE = mean(|y_true - y_pred|)
```

### mean_absolute_percentage_error

Scale-independent percentage error.

```
MAPE = mean(|y_true - y_pred| / |y_true|) * 100
```

### mean_squared_logarithmic_error

Useful when targets span several orders of magnitude.

```
MSLE = mean((log(1 + y_pred) - log(1 + y_true))^2)
```

### squared_hinge

Squared hinge loss for binary classification with labels in {-1, +1}.

```
squared_hinge = mean(max(0, 1 - y_true * y_pred)^2)
```

### hinge

Standard SVM-style hinge loss.

```
hinge = mean(max(0, 1 - y_true * y_pred))
```

### categorical_hinge

Multi-class hinge loss variant.

### categorical_crossentropy

Standard multi-class cross-entropy. Requires one-hot encoded targets and softmax output.

```
categorical_crossentropy = -sum(y_true * log(y_pred))
```

### sparse_categorical_crossentropy

Same as categorical_crossentropy but accepts integer class indices rather than one-hot vectors.

### binary_crossentropy

Binary cross-entropy for binary or multi-label classification with sigmoid output.

```
binary_crossentropy = -y_true * log(y_pred) - (1 - y_true) * log(1 - y_pred)
```

### kullback_leibler_divergence

Measures divergence between two distributions.

```
KLD = sum(y_true * log(y_true / y_pred))
```

Use for variational autoencoders and distributional matching.

### poisson

Poisson loss for count-based predictions.

```
poisson = mean(y_pred - y_true * log(y_pred))
```

### cosine_proximity

Negative cosine similarity. Used for metric learning and similarity tasks.

```
cosine_proximity = -sum(y_true * y_pred) / (||y_true|| * ||y_pred||)
```

---

## logcosh (Not Supported)

`logcosh` (log of the hyperbolic cosine of the prediction error) is not implemented in DL4J. Use `mean_absolute_error` as an alternative for outlier-robust regression tasks.

---

## Notes

Loss functions are read from the Keras training configuration embedded in the HDF5 file. They are only present when the model was compiled (`model.compile(...)`) and saved with `model.save()`. When loading for inference only, pass `enforceTrainingConfig=false` to bypass training configuration parsing.
