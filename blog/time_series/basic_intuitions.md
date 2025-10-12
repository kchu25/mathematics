@def title = " Time Series Analysis: Intuition and Implementation"
@def published = "12 October 2025"
@def tags = ["time-series"]

# Time Series Analysis: Intuition and Implementation

## The Big Picture

Time series is fundamentally about **patterns in time**. When you look at stock prices or daily temperatures, there's structure hidden in the chaos. Some of it repeats (like seasons), some of it drifts (like climate change), and some of it is just noise. Our job is to separate signal from noise and use the signal to predict what comes next.

The key insight that makes time series different: **today depends on yesterday**. In regular statistics, rows are independent—you can shuffle them without losing information. In time series, shuffle the data and you destroy the very thing you're trying to study. The temporal dependency *is* the data.

## Decomposing Reality

Think of any time series as having multiple voices singing at once. There's the slow, deep bass of the long-term trend. There's the predictable rhythm of seasonality. There might be irregular cycles. And there's always noise—the random static in the background.

Mathematically, we write this as either a sum or a product:
- Additive: $Y_t = T_t + S_t + C_t + R_t$
- Multiplicative: $Y_t = T_t \times S_t \times C_t \times R_t$

The additive model says "seasonality adds the same amount regardless of the trend level." Think: daily temperature swings are roughly the same in winter and summer. The multiplicative model says "seasonality scales with the trend." Think: retail sales where holiday bumps get bigger as the company grows.

Let's extract the trend using a moving average—essentially smoothing out the bumps by averaging over a window:

$\text{MA}_t = \frac{1}{k}\sum_{i=0}^{k-1} y_{t-i}$

where $k$ is the window size.

```julia
function moving_average(data, window)
    n = length(data)
    trend = fill(NaN, n)
    for i in window:n
        trend[i] = mean(data[i-window+1:i])
    end
    return trend
end
```

This is conceptually simple: to see the forest, blur the trees. The larger your window, the smoother your trend, but the more you lag behind changes.

## Stationarity: The Foundation

Here's the most important concept in time series: **stationarity**. A stationary series has no trend, no seasonality that changes over time, and constant variance. Its statistical properties don't depend on *when* you look at it.

Why does this matter? Because almost every time series model assumes stationarity. Think about it: if the mean keeps drifting upward, how can you estimate "the" mean? If variance explodes over time, how can you quantify uncertainty? You can't build a stable model on shifting sand.

The good news: we can often *transform* non-stationary data into stationary data. The most common trick is differencing:

$\nabla y_t = y_t - y_{t-1}$

This removes linear trends. If that's not enough, difference again (second-order differencing removes quadratic trends). For multiplicative growth, take logs first.

```julia
function difference(data, order=1)
    result = copy(data)
    for _ in 1:order
        result = diff(result)  # Applies ∇ operator
    end
    return result
end
```

When you plot your differenced series, you should see it hovering around a constant mean with roughly constant variance. That's your signal that you've achieved stationarity.

> **Important caveat:** Yes, you analyze and model the differenced data, but interpretation changes completely. A model on differenced data predicts *changes*, not levels. If you fit ARIMA(1,1,0), the "1,1,0" means you differenced once, so your model forecasts tomorrow's *change* from today. To get actual predictions, you must **integrate back**—accumulate those predicted changes starting from your last observed value. This is why ARIMA has that "I" (Integrated): it automatically handles the integration step when forecasting. The danger: if you forget to integrate back, you're making predictions on the wrong scale. The upside: modeling changes is often easier and more stable than modeling levels, especially when the level is trending or exploding.

## Autocorrelation: Measuring Memory

Once your data is stationary, the next question is: how much memory does this process have? Does today strongly depend on yesterday? Or does the influence fade quickly?

The **autocorrelation function** (ACF) measures this. The ACF at lag $k$ tells you the correlation between $y_t$ and $y_{t-k}$:

$\rho_k = \frac{\sum_{t=k+1}^{n}(y_t - \bar{y})(y_{t-k} - \bar{y})}{\sum_{t=1}^{n}(y_t - \bar{y})^2} = \frac{\text{Cov}(y_t, y_{t-k})}{\text{Var}(y_t)}$

```julia
function acf(data, lag)
    n = length(data)
    μ = mean(data)
    
    # Covariance at lag k
    numerator = sum((data[i] - μ) * (data[i-lag] - μ) for i in lag+1:n)
    # Variance
    denominator = sum((x - μ)^2 for x in data)
    
    return numerator / denominator
end
```

When you plot ACF against lag, you see how the memory decays. Does it cut off sharply after a few lags? Does it decay slowly? This pattern is diagnostic—it tells you what kind of model will work.

The **partial autocorrelation function** (PACF) is subtler. It measures the correlation between $y_t$ and $y_{t-k}$ after removing the influence of all the lags in between. If ACF tells you about total dependence, PACF tells you about direct dependence.

## Building Models: The Autoregressive Approach

The simplest time series model says: today is a weighted average of the past few days, plus some noise. This is the **autoregressive** (AR) model:

$$y_t = c + \phi_1 y_{t-1} + \phi_2 y_{t-2} + \cdots + \phi_p y_{t-p} + \varepsilon_t$$

The AR(1) case is particularly intuitive. If $\phi_1 = 0.8$, then today is 80% of yesterday (plus some baseline and noise). The closer $\phi_1$ is to 1, the more persistent the process—shocks take longer to fade. If $\phi_1 > 1$, the process explodes (non-stationary). If $\phi_1 < 0$, you get oscillation.

Fitting an AR(1) is just linear regression:

$\begin{align}
\phi &= \frac{\sum_i (y_{t-1,i} - \bar{y}_{t-1})(y_{t,i} - \bar{y}_t)}{\sum_i (y_{t-1,i} - \bar{y}_{t-1})^2} \\
c &= \bar{y}_t - \phi \bar{y}_{t-1}
\end{align}$

```julia
function fit_ar1(data)
    y = data[2:end]
    y_lag = data[1:end-1]
    
    μ_y = mean(y)
    μ_lag = mean(y_lag)
    
    # Estimate slope φ
    φ = sum((y_lag[i] - μ_lag) * (y[i] - μ_y) for i in 1:length(y)) / 
        sum((x - μ_lag)^2 for x in y_lag)
    
    # Estimate intercept c
    c = μ_y - φ * μ_lag
    
    return c, φ
end
```

To forecast, you just iterate the model forward. One-step-ahead is easy: $\hat{y}_{t+1} = c + \phi y_t$. For two steps ahead, plug in your forecast: $\hat{y}_{t+2} = c + \phi \hat{y}_{t+1}$. Notice how uncertainty grows as you forecast further out.

$\hat{y}_{t+h} = c + \phi \hat{y}_{t+h-1} \quad \text{for } h = 1, 2, \ldots$

```julia
function predict_ar1(data, c, φ, steps=1)
    predictions = Float64[]
    last_val = data[end]
    
    for _ in 1:steps
        next_val = c + φ * last_val  # Iterative forecast
        push!(predictions, next_val)
        last_val = next_val
    end
    
    return predictions
end
```

## The Moving Average Perspective

AR models say "today depends on past values." But there's another equally valid perspective: "today depends on past forecast errors." This is the **moving average** (MA) model:

$$y_t = \mu + \varepsilon_t + \theta_1 \varepsilon_{t-1} + \theta_2 \varepsilon_{t-2} + \cdots + \theta_q \varepsilon_{t-q}$$

This might seem weird at first. How can today depend on yesterday's error if we don't know yesterday's error until after it happens? The key is that once we've observed $y_{t-1}$, we can compute the error $\varepsilon_{t-1} = y_{t-1} - \hat{y}_{t-1}$. The MA model says these errors have momentum—if you over-predicted yesterday, you might over-predict today too.

MA models are trickier to fit because the errors aren't directly observed. But there's a neat relationship: for MA(1), the ACF at lag 1 is $\rho_1 = \theta/(1+\theta^2)$, and it's zero for all higher lags. This gives us a way to estimate $\theta$:

Solving $\rho_1 = \frac{\theta}{1+\theta^2}$ for $\theta$ gives:
$\theta = \frac{1 - \sqrt{1-4\rho_1^2}}{2\rho_1}$

```julia
function fit_ma1(data)
    μ = mean(data)
    ρ₁ = acf(data, 1)
    
    # Solve: ρ₁ = θ/(1 + θ²)
    discriminant = 1 - 4ρ₁^2
    θ = discriminant ≥ 0 ? (1 - sqrt(discriminant)) / (2ρ₁) : 0.5
    
    return μ, θ
end
```

## Combining Perspectives: ARMA and ARIMA

Why choose between AR and MA when you can have both? The **ARMA** model combines them:

$$y_t = c + \phi_1 y_{t-1} + \cdots + \phi_p y_{t-p} + \theta_1 \varepsilon_{t-1} + \cdots + \theta_q \varepsilon_{t-q} + \varepsilon_t$$

This is more flexible—it can capture complex dependencies that pure AR or pure MA miss. The art is choosing $p$ and $q$ (the AR and MA orders). This is where ACF and PACF plots become diagnostic tools:

- If ACF cuts off sharply but PACF decays gradually, you've got an MA process
- If PACF cuts off sharply but ACF decays gradually, you've got an AR process  
- If both decay gradually, you need ARMA

But what if your data isn't stationary? That's where **ARIMA** comes in—it's just ARMA with differencing built in. ARIMA($p$, $d$, $q$) means: difference $d$ times to achieve stationarity, then fit ARMA($p$, $q$).

The process: $y_t \xrightarrow{\nabla^d} w_t \xrightarrow{\text{ARMA}(p,q)} \hat{w}_t \xrightarrow{\text{integrate}} \hat{y}_t$

```julia
function arima_forecast(data, p, d, q, steps=1)
    # Step 1: Difference d times (∇^d operator)
    differenced = data
    for _ in 1:d
        differenced = diff(differenced)
    end
    
    # Step 2: Fit ARMA (simplified to AR(1) here)
    c, φ = fit_ar1(differenced)
    
    # Step 3: Forecast the differenced series
    diff_preds = predict_ar1(differenced, c, φ, steps)
    
    # Step 4: Integrate back (reverse ∇ operator)
    predictions = Float64[]
    last_val = data[end]
    for dp in diff_preds
        next_val = last_val + dp  # Integration: inverse of differencing
        push!(predictions, next_val)
        last_val = next_val
    end
    
    return predictions
end
```

The integration step is crucial. After differencing, your model predicts *changes*, not levels. To get back to the original scale, you accumulate those changes starting from your last observed value.

## Model Selection: Finding the Right Fit

How do you choose between AR(1), AR(2), ARMA(1,1), etc.? You need a criterion that balances fit quality against complexity. The **Akaike Information Criterion** (AIC) does exactly this:

$$\text{AIC} = n \ln\left(\frac{\text{SSE}}{n}\right) + 2k$$

where $k$ is the number of parameters. Lower AIC is better. The first term rewards good fit (low sum of squared errors), but the second term penalizes complexity. It's Occam's razor in equation form.

```julia
function aic(residuals, num_params)
    n = length(residuals)
    sse = sum(r^2 for r in residuals)
    return n * log(sse / n) + 2num_params
end
```

Fit several models, compute AIC for each, and pick the lowest. But don't stop there—always check the residuals.

## Exponential Smoothing: A Different Philosophy

ARIMA models are theoretically elegant, but there's another approach that's incredibly practical: **exponential smoothing**. The idea is beautifully simple: recent observations matter more than old ones, and the importance decays exponentially.

Simple exponential smoothing gives you a running weighted average:

$s_t = \alpha y_t + (1-\alpha) s_{t-1}$

The parameter $\alpha$ controls the memory. High $\alpha$ (close to 1) means "forget the past quickly, track recent changes." Low $\alpha$ means "smooth out noise, change slowly."

```julia
function simple_exp_smooth(data, α)
    n = length(data)
    smoothed = zeros(n)
    smoothed[1] = data[1]
    
    for i in 2:n
        smoothed[i] = α * data[i] + (1 - α) * smoothed[i-1]  # Weighted average
    end
    
    return smoothed
end
```

You can expand this to handle trends. **Holt's method** tracks both level and trend with separate smoothing equations:

$\begin{align}
\ell_t &= \alpha y_t + (1-\alpha)(\ell_{t-1} + b_{t-1}) \\
b_t &= \beta(\ell_t - \ell_{t-1}) + (1-\beta) b_{t-1}
\end{align}$

The level equation says "today's level is a blend of today's observation and yesterday's level+trend." The trend equation says "today's trend is a blend of today's level change and yesterday's trend."

```julia
function holt_linear(data, α, β)
    n = length(data)
    level = zeros(n)
    trend = zeros(n)
    
    level[1] = data[1]
    trend[1] = data[2] - data[1]
    
    for i in 2:n
        # Update level: blend observation with prediction
        level[i] = α * data[i] + (1 - α) * (level[i-1] + trend[i-1])
        # Update trend: blend current change with past trend
        trend[i] = β * (level[i] - level[i-1]) + (1 - β) * trend[i-1]
    end
    
    return level, trend
end
```

Forecasting with Holt is just extrapolating the latest level and trend linearly. It's simple, interpretable, and often works surprisingly well.

$\hat{y}_{t+h} = \ell_t + h \cdot b_t$

```julia
function forecast_holt(data, α, β, steps)
    level, trend = holt_linear(data, α, β)
    return [level[end] + h * trend[end] for h in 1:steps]  # Linear extrapolation
end
```

## Validation: Trust but Verify

You've fit a model—great! But is it any good? The first check is always the residuals. If your model is capturing all the structure, what's left should be white noise: random, zero mean, constant variance, no autocorrelation.

$\varepsilon_t = y_t - \hat{y}_t \quad \text{should be} \sim \mathcal{N}(0, \sigma^2) \text{ i.i.d.}$

```julia
function check_residuals(residuals)
    μ = mean(residuals)
    σ² = var(residuals)
    
    # Mean should be near zero: |μ| << σ
    mean_ok = abs(μ) < 0.1 * sqrt(σ²)
    
    # No autocorrelation: ρ₁ ≈ 0
    acf1 = acf(residuals, 1)
    acf_ok = abs(acf1) < 0.2
    
    return (mean=μ, variance=σ², mean_ok=mean_ok, 
            acf1=acf1, acf_ok=acf_ok)
end
```

If residuals show patterns, your model missed something. Maybe you need a higher order AR term, or seasonal adjustment, or a transformation.

For out-of-sample validation, you can't use random train/test splits—you'd be cheating by using future information. Instead, use rolling windows: train on days 1-100, test on 101-110. Then train on 11-110, test on 111-120. And so on.

**Rolling window CV:** Train on $[t, t+n]$, test on $[t+n+1, t+n+m]$, slide forward.

```julia
function time_series_cv(data, train_size, test_size)
    folds = []
    for i in 1:(length(data) - train_size - test_size + 1)
        train = data[i:i+train_size-1]
        test = data[i+train_size:i+train_size+test_size-1]
        push!(folds, (train, test))
    end
    return folds
end
```

## Measuring Forecast Quality

Once you have predictions and actuals, how do you score them? Three common metrics:

**MAE** (Mean Absolute Error) is intuitive—average size of your mistakes:
$$\text{MAE} = \frac{1}{n}\sum_{i=1}^{n}|y_i - \hat{y}_i|$$

**RMSE** (Root Mean Squared Error) penalizes big errors more:
$$\text{RMSE} = \sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}$$

**MAPE** (Mean Absolute Percentage Error) is scale-free, good for comparing across series:
$$\text{MAPE} = \frac{100}{n}\sum_{i=1}^{n}\left|\frac{y_i - \hat{y}_i}{y_i}\right|$$

```julia
mae(actual, predicted) = mean(abs.(actual .- predicted))
rmse(actual, predicted) = sqrt(mean((actual .- predicted).^2))
mape(actual, predicted) = 100 * mean(abs.((actual .- predicted) ./ actual))
```

Each has strengths and weaknesses. MAE is robust to outliers. RMSE is differentiable (nice for optimization). MAPE fails when actuals are near zero. Use multiple metrics to get a complete picture.

## Practical Wisdom

The workflow is rarely linear. You plot, notice non-stationarity, difference, replot, fit a model, check residuals, realize you have seasonality you missed, go back and deseasonalize, refit, check again. It's iterative.

Start simple. A moving average baseline often performs surprisingly well. Only add complexity when simple models fail. An ARIMA(5,2,4) that barely beats ARIMA(1,1,1) probably isn't worth the extra parameters—you're fitting noise.

Domain knowledge matters enormously. No amount of statistical sophistication will save you if you don't understand what you're modeling. Why would sales spike in Q4? Is this sensor data affected by temperature? Are there known policy changes? Let context guide your model choices.

And remember: all forecasts are wrong, but some are useful. The goal isn't perfection—it's capturing enough structure to make better decisions than you would without the model. Quantify uncertainty, communicate clearly, and update as new data arrives.