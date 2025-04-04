title: DataFrame output API in scikit-learn
use_katex: False
class: title-slide

# DataFrame Output API in scikit-learn

.g.g-middle[
.g-6[
![:scale 70%](images/scikit-learn-logo-without-subtitle.svg)
]
.g-6[
![:scale 70%](images/pandas-logo.png)
]
]

.larger[Thomas J. Fan]<br>
<a href="https://www.github.com/thomasjpfan" target="_blank" class="title-link"><span class="icon icon-github right-margin"></span>@thomasjpfan</a>
<a class="this-talk-link", href="https://github.com/thomasjpfan/sklearn-set-output" target="_blank">github.com/thomasjpfan/sklearn-set-output</a>

---

# DataFrame Output API 💻

.g[
.g-6[
## Default
```python
X = pd.DataFrame({
    "age": [10, 23, 43, 19],
    "height": [65, 60, 58, 62]
})

scalar = StandardScaler()
scalar.fit_transform(X)
# array([[-1.1391779 ,  1.45010473],
#        [-0.06213698, -0.48336824],
#        [ 1.59484907, -1.25675744],
#        [-0.39353419,  0.29002095]])
```
]
.g-6[
]
]

---

# DataFrame Output API 💻

.g[
.g-6[
## Default
```python
X = pd.DataFrame({
    "age": [10, 23, 43, 19],
    "height": [65, 60, 58, 62]
})

scalar = StandardScaler()
scalar.fit_transform(X)
# array([[-1.1391779 ,  1.45010473],
#        [-0.06213698, -0.48336824],
#        [ 1.59484907, -1.25675744],
#        [-0.39353419,  0.29002095]])
```
]
.g-6[

## `set_output` API

```python
scalar.set_output(transform="pandas")
scalar.transform(X)
```
![:scale 50%](images/df-out.jpg)
]
]


---

# API Design Journey 🛣️

1. Input Validation with `feature_names_in_`
2. `get_feature_names_out`
3. Three API Options for DataFrame out

---

class: top

# Input Validation with `feature_names_in_` API

```python
print(scalar.feature_names_in_)
# ['age' 'height']
```

--

### Column name consistency

```python
X2 = pd.DataFrame({"lat": [1, 2], "long": [-2, -4]})
try:
*   scalar.transform(X2)
except ValueError as e:
    print(e)
```

--

```
The feature names should match those that were passed during fit.
Feature names unseen at fit time:
- lat
- long
Feature names seen at fit time, yet now missing:
- age
- height
```

--

**Implementation**: Deprecated old behavior

---

class: top
<br>

# `get_feature_names_out` API 🌎

Mapping from input feature names to output names

--

### One to one

```python
feature_names_out = scalar.get_feature_names_out(X.columns)
print(feature_names_out)
# ['age' 'height']
```

--

### Independent of input features

```python
pca = PCA(n_components=5)
pca.fit(X)

print(pca.get_feature_names_out())
# ['pca0' 'pca1' 'pca2' 'pca3' 'pca4']
```

--

**Implementation**: Deprecated `get_feature_names`

---

# Three API Options for DataFrame out 🧪

1. DataFrame in & DataFrame out: [Rejected SLEP014](https://scikit-learn-enhancement-proposals.readthedocs.io/en/latest/slep014/proposal.html)
2. Custom `InputArray`: [Rejected SLEP012](https://scikit-learn-enhancement-proposals.readthedocs.io/en/latest/slep012/proposal.html)
3. `set_output` API: [Accepted SLEP018](https://scikit-learn-enhancement-proposals.readthedocs.io/en/latest/slep018/proposal.html)

---

class: top
<br><br><br>

# Option 1: DataFrame in & DataFrame out

```python
X = pd.DataFrame(...)
X_out = scalar.fit_transform(X)

type(X_out)
# pandas.core.frame.DataFrame
```

--

- **Pros**: Simple
- **Cons**: Not backward compatible

---

class: top
<br><br>

# Option 2: Custom `InputArray`

```python
*X = make_input_array(X, feature_names)

X_out = scalar.fit_transform(X)
type(X_out)
# InputArray
```

--

### Convert to Pandas

```
X_df = X_out.to_dataframe()
type(X_df)
# pandas.core.frame.DataFrame
```

--

- **Pros**: Opportunity to optimize numpy & sparse data
- **Cons**: Adding another "Dataframe-like container" into the ecosystem

---

class: top
<br><br><br>

# Option 3: `set_output` API

```python
X_df = pd.DataFrame(...)

*scalar.set_output(transform="pandas")
X_out_df = scalar.fit_transform(X_df)

type(X_out_df)
# pandas.core.frame.DataFrame
```

--

- **Pros**: Very explicit
- **Cons**: Does not support sparse data: pandas only represents CSC sparse data

---

class: top
<br><br>

# Option 3: `set_output` API

### Global config

```python
*sklearn.set_config(transform_output="pandas")

X_out_df = scalar.fit_transform(X_df)
```

--

### Polars output

```python
*sklearn.set_config(transform_output="polars")

X_out_df = scalar.transform(X)
type(X_out_df)
# polars.dataframe.frame.DataFrame
```

---

class: top
<br><br>

# `set_output` Future 🔮

## Predict output

```python
classifier.set_output(predict_proba="pandas")
```

--

## XArray output

```python
scalar.set_output(transform="xarray")
```

--

## Custom data containers

```python
sklearn.register_output(CustomDataFrameAdapter())
```
