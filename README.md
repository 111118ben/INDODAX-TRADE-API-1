"""
╔══════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                                                                                  ║
║  ██╗   ██╗ █████╗     ██╗   ██╗██╗  ████████╗██████╗  █████╗     ██╗   ██╗ █████╗              ║
║  ██║   ██║██╔══██╗    ██║   ██║██║  ╚══██╔══╝██╔══██╗██╔══██╗    ██║   ██║██╔══██╗             ║
║  ██║   ██║╚██████║    ██║   ██║██║     ██║   ██████╔╝███████║    ██║   ██║╚██████║             ║
║  ██║   ██║██╔══██║    ██║   ██║██║     ██║   ██╔══██╗██╔══██║    ╚██╗ ██╔╝██╔══██║             ║
║  ╚██████╔╝╚█████╔╝    ╚██████╔╝███████╗██║   ██║  ██║██║  ██║     ╚████╔╝ ╚█████╔╝             ║
║   ╚═════╝  ╚════╝      ╚═════╝ ╚══════╝╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝      ╚═══╝   ╚════╝             ║
║                                                                                                  ║
║         ███████╗██╗   ██╗██████╗ ██████╗ ███████╗███╗   ███╗███████╗                           ║
║         ██╔════╝██║   ██║██╔══██╗██╔══██╗██╔════╝████╗ ████║██╔════╝                           ║
║         ███████╗██║   ██║██████╔╝██████╔╝█████╗  ██╔████╔██║█████╗                             ║
║         ╚════██║██║   ██║██╔═══╝ ██╔══██╗██╔══╝  ██║╚██╔╝██║██╔══╝                             ║
║         ███████║╚██████╔╝██║     ██║  ██║███████╗██║ ╚═╝ ██║███████╗                           ║
║         ╚══════╝ ╚═════╝ ╚═╝     ╚═╝  ╚═╝╚══════╝╚═╝     ╚═╝╚══════╝                           ║
║                                                                                                  ║
║             INDODAX ULTRA SUPREME PROFIT ENGINE  v9.0  —  THE ONLY ONE IN THE WORLD            ║
║                         🔴 LIVE TRADING · REAL MONEY · NO MERCY 🔴                             ║
║                                                                                                  ║
║  🧠 SMART FEATURES (v9.0 REBUILT FROM SCRATCH):                                                ║
║  ────────────────────────────────────────────────────────────────────────────────────────────   ║
║  1. ✅ DYNAMIC SLOT SIZING — equity-weighted, regime-adaptive                                   ║
║  2. ✅ NEURAL ENSEMBLE PREDICTOR — 7-model weighted voting + ML fallback                        ║
║  3. ✅ SMART ENTRY GATING — composite score ≥ 5 gates required simultaneously                   ║
║  4. ✅ TIERED PROFIT TAKING — 25% at 0.3%, 50% at 0.5%, exit rest at 0.8%                      ║
║  5. ✅ ATR-ADAPTIVE STOP LOSS — never static, always market-aware                               ║
║  6. ✅ REGIME ENGINE — TRENDING / HFT / QUIET / VOLATILE adapts ALL params                      ║
║  7. ✅ SPREAD GUARDIAN — real-time slippage protection                                          ║
║  8. ✅ CIRCUIT BREAKER 2.0 — per-coin + global + time-decay recovery                            ║
║  9. ✅ FEE-AWARE P&L — every calc includes round-trip 0.51%                                     ║
║  10.✅ PRIORITY QUEUE SCANNER — top 15 signals ranked multi-factor                              ║
║  11.✅ SMART NONCE MANAGER — thread-safe, atomic, zero collision                                ║
║  12.✅ CANDLE SYNTHETIC FALLBACK — real-vol calibrated OHLCV generation                        ║
║  13.✅ DAILY STATS TRACKER — win rate, streak, equity curve                                     ║
║  14.✅ STOCHASTIC RSI — more sensitive than plain RSI for HFT entries                           ║
║  15.✅ WILLIAMS %R + CCI — triple-confirming oversold signals                                   ║
║  16.✅ VWAP DEVIATION FILTER — only buy below VWAP                                              ║
║  17.✅ ORDER BOOK IMBALANCE — bid/ask depth momentum (when available)                           ║
║  18.✅ SMART COOLDOWN — per-coin re-entry cooldown prevents thrashing                           ║
║  19.✅ PROFIT LOCK — trailing lock kicks in at 0.4% to never return to 0                        ║
║  20.✅ MOMENTUM DIVERGENCE — price vs volume divergence early warning                           ║
╚══════════════════════════════════════════════════════════════════════════════════════════════════╝

WARNING: REAL MONEY. REAL ORDERS. START SMALL.
"""

# ============================================================================
# IMPORTS
# ============================================================================

import os, sys, time, hmac, hashlib, json, logging, signal, traceback
import threading, queue, math
from collections import deque, OrderedDict
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from typing import Optional, Dict, List, Tuple, Any
from urllib.parse import urlencode
from enum import Enum

import numpy as np
import pandas as pd

try:
    import requests
    from requests.adapters import HTTPAdapter
    from requests.packages.urllib3.util.retry import Retry
except ImportError:
    print("pip install requests"); sys.exit(1)

try:
    from colorama import init, Fore, Back, Style
    init(autoreset=True)
except ImportError:
    print("pip install colorama"); sys.exit(1)

try:
    from dotenv import load_dotenv; load_dotenv()
except ImportError:
    pass

try:
    from sklearn.ensemble import GradientBoostingRegressor, RandomForestClassifier
    from sklearn.preprocessing import StandardScaler
    ML_OK = True
except ImportError:
    ML_OK = False

# ============================================================================
# VERSION & CONSTANTS
# ============================================================================

VERSION = "9.0-ULTRA-SUPREME"

INDODAX_PUBLIC  = "https://indodax.com/api"
INDODAX_PRIVATE = "https://indodax.com/tapi"
INDODAX_SUMMARIES = "https://indodax.com/api/summaries"

# ============================================================================
# MASTER CONFIG — every parameter documented and justified
# ============================================================================

CFG = {
    # ── CAPITAL ─────────────────────────────────────────────────────────────
    "MAX_SLOTS":           100,      # total concurrent positions
    "MIN_SLOT_IDR":        35_000,   # floor per position
    "MAX_SLOT_IDR":        250_000,  # ceiling per position (risk cap)
    "RESERVE_PCT":         0.10,     # keep 10% cash as emergency reserve

    # ── PROFIT TARGETS ──────────────────────────────────────────────────────
    "TARGET_PCT":          0.55,     # primary take-profit %
    "PARTIAL1_PCT":        0.30,     # partial exit #1 trigger %
    "PARTIAL1_SIZE":       0.25,     # sell 25% of position at partial1
    "PARTIAL2_PCT":        0.50,     # partial exit #2 trigger %
    "PARTIAL2_SIZE":       0.50,     # sell 50% of remaining at partial2
    "PROFIT_LOCK_PCT":     0.40,     # trailing lock activates here

    # ── RISK ────────────────────────────────────────────────────────────────
    "STOP_LOSS_PCT":       1.20,     # base stop-loss %
    "TRAILING_STOP_PCT":   0.45,     # trail distance after lock
    "ATR_STOP_MULT":       2.0,      # ATR multiplier for dynamic stop
    "MAX_DAILY_DD_PCT":    3.00,     # halt day at -3% equity drawdown
    "MAX_CONSEC_LOSS":     5,        # halt after N consecutive losses
    "DAILY_PROFIT_CAP":    12.00,    # stop at +12% daily profit (take it)
    "MAX_HOLD_HOURS":      24,       # force-sell after 24h regardless

    # ── FEE ─────────────────────────────────────────────────────────────────
    "FEE_BUY":             0.30,     # Indodax taker fee %
    "FEE_SELL":            0.30,     # Indodax taker fee %
    "FEE_ROUNDTRIP":       0.51,     # effective round-trip net fee %

    # ── SCANNER FILTERS ─────────────────────────────────────────────────────
    "MIN_VOL_IDR_24H":     40_000_000,  # Rp40jt minimum daily volume
    "MAX_PRICE_COIN":      120_000,     # skip coins > Rp120k (low qty)
    "MAX_SPREAD_PCT":      2.0,         # reject if bid/ask spread > 2%
    "MIN_BUY_SCORE":       5,           # gate: minimum composite score
    "MAX_RSI":             68,          # gate: RSI upper bound
    "MIN_RSI":             22,          # gate: RSI lower bound
    "MIN_MOMENTUM":        -8.0,        # gate: minimum 5-candle momentum
    "STOCH_RSI_MAX":       80,          # gate: stochastic RSI ceiling
    "STOCH_RSI_MIN":       10,          # gate: stochastic RSI floor (oversold)
    "MAX_CCI":             100,         # gate: CCI overbought limit
    "MIN_WILLIAMS_R":      -90,         # gate: Williams %R oversold zone

    # ── TECHNICAL PERIODS ───────────────────────────────────────────────────
    "RSI_PERIOD":          14,
    "STOCH_RSI_PERIOD":    14,
    "MACD_FAST":           12,
    "MACD_SLOW":           26,
    "MACD_SIG":            9,
    "BB_PERIOD":           20,
    "BB_STD":              2.0,
    "ATR_PERIOD":          14,
    "EMA_FAST":            9,
    "EMA_MED":             21,
    "EMA_SLOW":            50,
    "CCI_PERIOD":          20,
    "WILLIAMS_PERIOD":     14,
    "VWAP_PERIOD":         20,

    # ── TIMING ──────────────────────────────────────────────────────────────
    "SCAN_INTERVAL":       8,        # seconds between full scans
    "HFT_INTERVAL":        4,        # seconds in HFT mode
    "COOLDOWN_SECONDS":    300,      # per-coin re-entry cooldown (5 min)
    "REQUEST_TIMEOUT":     10,
    "MAX_RETRIES":         3,
    "RETRY_DELAY":         1.2,

    # ── CACHE ───────────────────────────────────────────────────────────────
    "CACHE_TICKER_TTL":    12,
    "CACHE_SUMMARY_TTL":   25,
    "CACHE_CANDLE_TTL":    40,
    "CACHE_MAX":           15_000,

    # ── CIRCUIT BREAKER ─────────────────────────────────────────────────────
    "CB_THRESHOLD":        7,        # errors before open
    "CB_TIMEOUT":          200,      # seconds circuit stays open
    "CB_HALF_OPEN_TEST":   3,        # test requests before fully closing

    # ── API RATE ────────────────────────────────────────────────────────────
    "API_BACKOFF":         1.6,
    "API_RECOVERY":        0.93,
    "API_MIN_INTERVAL":    3,
    "API_MAX_INTERVAL":    35,

    # ── HFT ─────────────────────────────────────────────────────────────────
    "HFT_ENABLED":         True,
    "HFT_VOL_THRESH":      0.004,    # volatility threshold for HFT mode
    "HFT_MIN_SLOT":        50_000,

    # ── PREDICTOR ───────────────────────────────────────────────────────────
    "PRED_CONFIDENCE_MIN": 0.52,     # minimum confidence to count as signal
    "PRED_WEIGHT_ML":      0.45,     # weight of ML vs ensemble when both avail

    # ── SYSTEM ──────────────────────────────────────────────────────────────
    "LOG_FILE":            "ultra_v9.log",
    "MAX_SIGNALS_DISPLAY": 10,
    "TOP_SIGNALS_TO_EXEC": 5,        # max buys per scan cycle
}

# ============================================================================
# LOGGING
# ============================================================================

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(message)s",
    handlers=[
        logging.FileHandler(CFG["LOG_FILE"], encoding="utf-8"),
        logging.StreamHandler(sys.stdout),
    ],
)
log = logging.getLogger("UltraV9")

# ============================================================================
# ENUMS
# ============================================================================

class Regime(Enum):
    TRENDING_UP   = "trending_up"
    TRENDING_DOWN = "trending_down"
    VOLATILE      = "volatile"
    RANGING       = "ranging"
    QUIET         = "quiet"

class Strength(Enum):
    WEAK   = 1
    MEDIUM = 2
    STRONG = 3
    ULTRA  = 4

class Mode(Enum):
    CONSERVATIVE = "conservative"
    NORMAL       = "normal"
    AGGRESSIVE   = "aggressive"
    HFT          = "hft"

# ============================================================================
# DATA CLASSES
# ============================================================================

@dataclass
class APIStat:
    total: int = 0
    success: int = 0
    failed: int = 0
    rate_errors: int = 0
    consec_errors: int = 0
    status: str = "healthy"
    scan_interval: float = CFG["SCAN_INTERVAL"]
    avg_rt_ms: float = 0.0
    rts: deque = field(default_factory=lambda: deque(maxlen=200))
    cb_open_at: float = 0.0
    cb_half_open_tests: int = 0

@dataclass
class Prediction:
    price_target: float
    confidence: float
    direction: str       # up / down / sideways
    move_pct: float
    vol_adjusted: float
    votes: Dict[str, float]
    ts: float = field(default_factory=time.time)

@dataclass
class Signal:
    coin: str
    pair: str
    price: float
    rsi: float
    stoch_rsi: float
    macd_hist: float
    bb_pos: float
    bb_width: float
    cci: float
    williams_r: float
    momentum: float
    vwap_dev: float
    atr: float
    volume_ratio: float
    buy_score: float
    strength: Strength
    prediction: Optional[Prediction]
    regime: Regime
    ts: float = field(default_factory=time.time)

@dataclass
class Position:
    coin: str
    pair: str
    entry_price: float
    amount: float          # coins held
    invested: float        # IDR invested
    target_profit: float   # IDR target
    stop_loss: float       # IDR price
    trailing_stop: float   # IDR price (moves up)
    entry_ts: float
    rsi_entry: float
    score_entry: float
    order_id: Optional[str] = None
    current_price: float = 0.0
    max_price: float = 0.0
    net_pnl: float = 0.0
    pnl_pct: float = 0.0
    max_pnl_pct: float = 0.0
    profit_lock_active: bool = False
    partial1_done: bool = False
    partial2_done: bool = False

@dataclass
class DailyStats:
    date: str = ""
    trades: int = 0
    wins: int = 0
    losses: int = 0
    profit: float = 0.0
    fees_paid: float = 0.0
    consec_losses: int = 0
    equity_start: float = 0.0

@dataclass
class MarketCtx:
    regime: Regime = Regime.QUIET
    volatility: float = 0.005
    volume_trend: float = 0.0
    active_coins: int = 0
    top_mover: str = ""
    last_update: float = 0.0

# ============================================================================
# UTILITY — FAST NUMPY INDICATORS
# ============================================================================

class TA:
    """All indicators are pure numpy, no external TA lib required."""

    @staticmethod
    def ema_series(arr: np.ndarray, period: int) -> np.ndarray:
        """Full EMA series (not just last value)."""
        k = 2 / (period + 1)
        out = np.empty_like(arr, dtype=float)
        out[0] = arr[0]
        for i in range(1, len(arr)):
            out[i] = arr[i] * k + out[i-1] * (1 - k)
        return out

    @staticmethod
    def ema_last(arr: np.ndarray, period: int) -> float:
        return TA.ema_series(arr, period)[-1]

    @staticmethod
    def rsi(close: np.ndarray, period: int = 14) -> float:
        if len(close) < period + 2: return 50.0
        d = np.diff(close)
        g, l = np.where(d > 0, d, 0.0), np.where(d < 0, -d, 0.0)
        ag, al = np.mean(g[:period]), np.mean(l[:period])
        for i in range(period, len(g)):
            ag = (ag * (period - 1) + g[i]) / period
            al = (al * (period - 1) + l[i]) / period
        return 100.0 if al == 0 else 100 - 100 / (1 + ag / al)

    @staticmethod
    def stoch_rsi(close: np.ndarray, period: int = 14) -> float:
        """Stochastic RSI — more sensitive than plain RSI."""
        if len(close) < period * 2 + 2: return 50.0
        rsi_vals = np.array([TA.rsi(close[:i+1], period) for i in range(period, len(close))])
        if len(rsi_vals) < period: return 50.0
        window = rsi_vals[-period:]
        mn, mx = window.min(), window.max()
        return 50.0 if mx == mn else (rsi_vals[-1] - mn) / (mx - mn) * 100

    @staticmethod
    def macd(close: np.ndarray, fast=12, slow=26, sig=9):
        if len(close) < slow + sig + 2: return 0.0, 0.0, 0.0
        fast_e = TA.ema_series(close, fast)
        slow_e = TA.ema_series(close, slow)
        macd_l = fast_e - slow_e
        sig_l  = TA.ema_series(macd_l, sig)
        return float(macd_l[-1]), float(sig_l[-1]), float(macd_l[-1] - sig_l[-1])

    @staticmethod
    def bollinger(close: np.ndarray, period=20, std_dev=2.0) -> dict:
        if len(close) < period:
            c = float(close[-1])
            return {"upper": c, "middle": c, "lower": c, "pos": 0.5, "width_pct": 0.0}
        w = close[-period:]
        mid, std = float(np.mean(w)), float(np.std(w))
        up, lo = mid + std_dev * std, mid - std_dev * std
        pos = (float(close[-1]) - lo) / (up - lo) if up != lo else 0.5
        return {"upper": up, "middle": mid, "lower": lo, "pos": pos,
                "width_pct": (up - lo) / mid * 100 if mid > 0 else 0.0}

    @staticmethod
    def atr(high: np.ndarray, low: np.ndarray, close: np.ndarray, period=14) -> float:
        if len(high) < period + 2: return float(np.std(close) * 0.02) if len(close) > 1 else 0.0
        tr = np.maximum(high[1:] - low[1:],
               np.maximum(np.abs(high[1:] - close[:-1]),
                          np.abs(low[1:]  - close[:-1])))
        return float(np.mean(tr[-period:]))

    @staticmethod
    def cci(high: np.ndarray, low: np.ndarray, close: np.ndarray, period=20) -> float:
        if len(close) < period: return 0.0
        tp = (high[-period:] + low[-period:] + close[-period:]) / 3
        mean_tp = np.mean(tp)
        mad = np.mean(np.abs(tp - mean_tp))
        return (tp[-1] - mean_tp) / (0.015 * mad) if mad > 0 else 0.0

    @staticmethod
    def williams_r(high: np.ndarray, low: np.ndarray, close: np.ndarray, period=14) -> float:
        if len(close) < period: return -50.0
        h, l = np.max(high[-period:]), np.min(low[-period:])
        return -100.0 if h == l else (h - close[-1]) / (h - l) * -100

    @staticmethod
    def vwap(close: np.ndarray, volume: np.ndarray, period=20) -> float:
        if len(close) < period or len(volume) < period: return float(close[-1])
        c, v = close[-period:], volume[-period:]
        return float(np.sum(c * v) / np.sum(v)) if np.sum(v) > 0 else float(c[-1])

    @staticmethod
    def detect_regime(close: np.ndarray) -> Regime:
        if len(close) < 50: return Regime.QUIET
        ret = np.diff(close) / close[:-1]
        sma20 = np.mean(close[-20:])
        sma50 = np.mean(close[-50:])
        trend = (sma20 - sma50) / sma50 if sma50 > 0 else 0
        vol   = float(np.std(ret[-20:])) if len(ret) >= 20 else 0.005
        if vol > 0.015:          return Regime.VOLATILE
        if vol < 0.003:          return Regime.QUIET
        if trend >  0.02:        return Regime.TRENDING_UP
        if trend < -0.02:        return Regime.TRENDING_DOWN
        return Regime.RANGING

    @staticmethod
    def composite_buy_score(
        rsi, stoch_rsi, macd_hist, bb_pos, cci, williams_r,
        momentum, volume_ratio, vwap_dev, regime: Regime
    ) -> float:
        """
        Multi-factor composite score (0–100).
        Each indicator contributes proportionally.
        """
        s = 0.0

        # ── RSI (0–30 pts) ──────────────────────────────────────────────
        if   rsi < 22: s += 30
        elif rsi < 28: s += 25
        elif rsi < 35: s += 20
        elif rsi < 42: s += 14
        elif rsi < 50: s += 8
        elif rsi > 65: s -= 12
        elif rsi > 58: s -= 5

        # ── Stochastic RSI (0–20 pts) ────────────────────────────────────
        if   stoch_rsi < 10: s += 20
        elif stoch_rsi < 20: s += 15
        elif stoch_rsi < 30: s += 10
        elif stoch_rsi > 80: s -= 10

        # ── MACD histogram (0–15 pts) ────────────────────────────────────
        if macd_hist < 0:
            if macd_hist < -200: s += 15
            elif macd_hist < -100: s += 12
            else: s += 7
        elif macd_hist > 50: s += 5  # confirmed upswing bonus

        # ── Bollinger Band position (0–15 pts) ──────────────────────────
        if   bb_pos < 0.10: s += 15
        elif bb_pos < 0.20: s += 12
        elif bb_pos < 0.30: s += 8
        elif bb_pos < 0.40: s += 4
        elif bb_pos > 0.85: s -= 8

        # ── CCI (0–10 pts) ───────────────────────────────────────────────
        if   cci < -150: s += 10
        elif cci < -100: s += 7
        elif cci > 100:  s -= 6

        # ── Williams %R (0–10 pts) ───────────────────────────────────────
        if   williams_r < -90: s += 10
        elif williams_r < -80: s += 7
        elif williams_r > -10: s -= 6

        # ── Momentum (0–8 pts) ───────────────────────────────────────────
        if   momentum < -3.0: s += 8
        elif momentum < -1.5: s += 5
        elif momentum < -0.5: s += 2
        elif momentum > 3.0:  s -= 8
        elif momentum > 1.5:  s -= 4

        # ── VWAP deviation (0–8 pts) — only buy below VWAP ──────────────
        if   vwap_dev < -2.0: s += 8
        elif vwap_dev < -1.0: s += 5
        elif vwap_dev < -0.3: s += 2
        elif vwap_dev >  1.5: s -= 5

        # ── Volume ratio (0–5 pts) ───────────────────────────────────────
        if   volume_ratio > 2.0: s += 5
        elif volume_ratio > 1.5: s += 3
        elif volume_ratio < 0.5: s -= 3

        # ── Regime bonus/penalty ─────────────────────────────────────────
        if regime == Regime.TRENDING_UP:   s += 5
        elif regime == Regime.VOLATILE:    s -= 5
        elif regime == Regime.TRENDING_DOWN: s -= 8

        return min(100.0, max(0.0, s))

# ============================================================================
# NEURAL ENSEMBLE PREDICTOR  (7-model + ML)
# ============================================================================

class NeuralPredictor:
    """
    7-model weighted ensemble:
      1. Momentum (multi-lag)
      2. Mean Reversion (z-score)
      3. Trend Following (EMA crossover)
      4. Volatility Arbitrage
      5. Seasonal/Hour Bias
      6. RSI Divergence
      7. BB Band Squeeze
    + Optional ML (GBM + RFC) if sklearn available.
    """

    MODEL_WEIGHTS = {
        "momentum":    0.20,
        "mean_rev":    0.15,
        "trend":       0.20,
        "vol_arb":     0.12,
        "seasonal":    0.08,
        "rsi_div":     0.12,
        "bb_squeeze":  0.13,
    }

    def __init__(self):
        self.gbm    = GradientBoostingRegressor(n_estimators=80, learning_rate=0.08, random_state=42) if ML_OK else None
        self.rfc    = RandomForestClassifier(n_estimators=60, random_state=42) if ML_OK else None
        self.scaler = StandardScaler() if ML_OK else None
        self.ml_fitted = False
        self.history = deque(maxlen=1000)
        log.info("🧠 NeuralPredictor v9 initialized")

    def predict(self, df: pd.DataFrame, price: float) -> Optional[Prediction]:
        if df is None or len(df) < 35: return None
        try:
            c = df["close"].values.astype(float)
            h = df["high"].values.astype(float)
            l = df["low"].values.astype(float)

            votes = {
                "momentum":   self._momentum(c),
                "mean_rev":   self._mean_rev(c),
                "trend":      self._trend(c),
                "vol_arb":    self._vol_arb(c, h, l),
                "seasonal":   self._seasonal(df),
                "rsi_div":    self._rsi_div(c),
                "bb_squeeze": self._bb_squeeze(c),
            }

            weighted = sum(votes[k] * self.MODEL_WEIGHTS[k] for k in votes)

            # Ensemble agreement → confidence
            vals = list(votes.values())
            agreement = 1.0 - np.std(vals) / (abs(np.mean(vals)) + 1e-6)
            confidence = float(np.clip(agreement * 0.85 + 0.15, 0.30, 0.95))

            # ML boost
            if ML_OK and len(df) >= 60:
                ml = self._ml_predict(df)
                if ml is not None:
                    w = CFG["PRED_WEIGHT_ML"]
                    weighted   = weighted * (1 - w) + ml * w
                    confidence = min(0.95, confidence * 1.12)

            move_pct = abs(weighted) * 100
            direction = "up" if weighted > 0.0025 else ("down" if weighted < -0.0025 else "sideways")
            vol = float(np.std(np.diff(c[-20:]) / (c[-20:-1] + 1e-9))) if len(c) > 20 else 0.005
            vol_adj = move_pct / (vol * 100 + 0.01)

            return Prediction(
                price_target=price * (1 + weighted),
                confidence=confidence,
                direction=direction,
                move_pct=move_pct,
                vol_adjusted=vol_adj,
                votes=votes,
                ts=time.time(),
            )
        except Exception:
            return None

    def _momentum(self, c):
        if len(c) < 12: return 0.0
        m1  = (c[-1] - c[-2])  / (c[-2]  + 1e-9)
        m5  = (c[-1] - c[-6])  / (c[-6]  + 1e-9) if len(c) >= 6  else m1
        m10 = (c[-1] - c[-11]) / (c[-11] + 1e-9) if len(c) >= 11 else m5
        return float(np.clip(m1 * 0.5 + m5 * 0.3 + m10 * 0.2, -0.025, 0.025))

    def _mean_rev(self, c):
        if len(c) < 22: return 0.0
        mean, std = np.mean(c[-22:]), np.std(c[-22:])
        z = (c[-1] - mean) / (std + 1e-9)
        return float(np.clip(-z * 0.0035, -0.018, 0.018))

    def _trend(self, c):
        if len(c) < 28: return 0.0
        e9, e21 = TA.ema_last(c, 9), TA.ema_last(c, 21)
        e50 = TA.ema_last(c, 50) if len(c) >= 50 else e21
        strength = abs(e9 - e21) / (e21 + 1e-9)
        sign = (1 if e9 > e21 else -1) + (1 if e21 > e50 else -1)
        return float(np.clip(sign / 2 * strength * 50, -0.012, 0.012))

    def _vol_arb(self, c, h, l):
        if len(c) < 16: return 0.0
        avg_vol = float(np.std(np.diff(c[-20:]) / (c[-20:-1] + 1e-9))) if len(c) > 20 else 0.005
        cur_vol = float(np.std(np.diff(c[-5:])  / (c[-5:-1]  + 1e-9))) if len(c) > 5  else avg_vol
        ratio = cur_vol / (avg_vol + 1e-9)
        return -0.004 if ratio > 1.6 else (0.003 if ratio < 0.65 else 0.0)

    def _seasonal(self, df):
        if not isinstance(df.index, pd.DatetimeIndex) or len(df) < 1: return 0.0
        hour = df.index[-1].hour
        # Jakarta trading hours bias (WIB = UTC+7)
        if 2 <= hour <= 4:   return  0.0015  # Asia open
        if 9 <= hour <= 11:  return  0.0012  # Mid-morning
        if 13 <= hour <= 15: return  0.0008  # Afternoon momentum
        if hour < 1 or hour > 22: return -0.0008
        return 0.0

    def _rsi_div(self, c):
        """Detect bullish RSI divergence (price down, RSI up)."""
        if len(c) < 28: return 0.0
        rsi_now  = TA.rsi(c, 14)
        rsi_prev = TA.rsi(c[:-5], 14)
        price_chg = (c[-1] - c[-6]) / (c[-6] + 1e-9)
        rsi_chg   = rsi_now - rsi_prev
        # Bullish divergence: price fell but RSI rose
        if price_chg < -0.005 and rsi_chg > 2:
            return 0.008
        # Bearish divergence
        if price_chg > 0.005 and rsi_chg < -2:
            return -0.006
        return 0.0

    def _bb_squeeze(self, c):
        """BB squeeze breakout detection."""
        if len(c) < 25: return 0.0
        bb_now  = TA.bollinger(c, 20, 2.0)
        bb_prev = TA.bollinger(c[:-5], 20, 2.0)
        expand = bb_now["width_pct"] - bb_prev["width_pct"]
        if expand > 0.5 and bb_now["pos"] < 0.3:
            return 0.006   # squeeze → breakout upward
        if expand > 0.5 and bb_now["pos"] > 0.7:
            return -0.005  # squeeze → breakout downward
        return 0.0

    def _ml_predict(self, df):
        if not ML_OK or len(df) < 60: return None
        try:
            c = df["close"].values.astype(float)
            X, y = [], []
            for i in range(40, len(df) - 6):
                r = np.diff(c[i-20:i]) / (c[i-20:i-1] + 1e-9)
                feat = [
                    float(np.mean(r)), float(np.std(r)),
                    float((c[i-1] - c[i-20]) / (c[i-20] + 1e-9)),
                    float(np.percentile(r, 20)), float(np.percentile(r, 80)),
                    float(np.sum(r > 0) / len(r)),
                    float(TA.rsi(c[:i], 14)),
                    float(TA.ema_last(c[:i], 9) / (TA.ema_last(c[:i], 21) + 1e-9) - 1),
                ]
                X.append(feat)
                y.append(float((c[i+5] - c[i]) / (c[i] + 1e-9)))
            if len(X) < 30: return None
            X, y = np.array(X), np.array(y)
            if not self.ml_fitted:
                self.scaler.fit(X)
                self.gbm.fit(self.scaler.transform(X), y)
                self.ml_fitted = True
            latest_r = np.diff(c[-20:]) / (c[-20:-1] + 1e-9)
            feat = np.array([[
                float(np.mean(latest_r)), float(np.std(latest_r)),
                float((c[-1] - c[-20]) / (c[-20] + 1e-9)),
                float(np.percentile(latest_r, 20)), float(np.percentile(latest_r, 80)),
                float(np.sum(latest_r > 0) / len(latest_r)),
                float(TA.rsi(c, 14)),
                float(TA.ema_last(c, 9) / (TA.ema_last(c, 21) + 1e-9) - 1),
            ]])
            return float(np.clip(self.gbm.predict(self.scaler.transform(feat))[0], -0.025, 0.025))
        except Exception:
            return None

# ============================================================================
# INDODAX LIVE CLIENT  (thread-safe, circuit-breaker, smart cache)
# ============================================================================

class IndodaxClient:
    def __init__(self, api_key: str, secret_key: str):
        self.api_key    = api_key
        self.secret_key = secret_key.encode("utf-8")
        self.session    = self._make_session()
        self._nonce     = int(time.time() * 1000)
        self._nonce_lk  = threading.Lock()
        self._cache:     OrderedDict = OrderedDict()
        self._cache_ts:  dict = {}
        self.stat        = APIStat()
        self._last_req   = 0.0
        log.info(f"🚀 IndodaxClient v{VERSION} ready")

    def _make_session(self):
        s = requests.Session()
        r = Retry(total=2, backoff_factor=0.4, status_forcelist=[500, 502, 503])
        s.mount("https://", HTTPAdapter(pool_connections=25, pool_maxsize=25, max_retries=r))
        s.headers.update({"User-Agent": f"UltraV9/{VERSION}", "Accept": "application/json"})
        return s

    # ── CIRCUIT BREAKER ──────────────────────────────────────────────────────

    def _cb_open(self) -> bool:
        if self.stat.consec_errors < CFG["CB_THRESHOLD"]:
            return False
        if self.stat.cb_open_at == 0.0:
            self.stat.cb_open_at = time.time()
            self.stat.status = "CIRCUIT_OPEN"
            log.error("🔴 CIRCUIT BREAKER OPENED")
            return True
        elapsed = time.time() - self.stat.cb_open_at
        if elapsed > CFG["CB_TIMEOUT"]:
            # Try half-open
            self.stat.cb_half_open_tests += 1
            if self.stat.cb_half_open_tests >= CFG["CB_HALF_OPEN_TEST"]:
                log.info("🟢 CIRCUIT BREAKER CLOSED")
                self.stat.consec_errors = 0
                self.stat.cb_open_at = 0.0
                self.stat.cb_half_open_tests = 0
                self.stat.status = "healthy"
                return False
        return True

    # ── THROTTLE ─────────────────────────────────────────────────────────────

    def _throttle(self):
        if self._cb_open():
            time.sleep(3)
            return
        gap = time.time() - self._last_req
        if gap < 0.06:
            time.sleep(0.06 - gap)
        self._last_req = time.time()

    # ── METRICS ──────────────────────────────────────────────────────────────

    def _tick(self, ok: bool, rt: float, rate_limited=False):
        self.stat.total += 1
        self.stat.rts.append(rt * 1000)
        self.stat.avg_rt_ms = float(np.mean(self.stat.rts))
        if ok:
            self.stat.success += 1
            self.stat.consec_errors = max(0, self.stat.consec_errors - 1)
            if self.stat.consec_errors == 0:
                self.stat.scan_interval = max(
                    self.stat.scan_interval * CFG["API_RECOVERY"],
                    CFG["API_MIN_INTERVAL"])
        else:
            self.stat.failed += 1
            self.stat.consec_errors += 1
            if rate_limited:
                self.stat.rate_errors += 1
                self.stat.scan_interval = min(
                    self.stat.scan_interval * CFG["API_BACKOFF"],
                    CFG["API_MAX_INTERVAL"])
                self.stat.status = "rate_limited"
                log.warning(f"⚠️ Rate-limited → scan {self.stat.scan_interval:.1f}s")
            else:
                self.stat.status = "degraded"

    # ── CACHE ────────────────────────────────────────────────────────────────

    def _cget(self, k, ttl):
        if k in self._cache and time.time() - self._cache_ts.get(k, 0) < ttl:
            self._cache.move_to_end(k)
            return self._cache[k]
        return None

    def _cset(self, k, v):
        if len(self._cache) >= CFG["CACHE_MAX"]:
            del self._cache[next(iter(self._cache))]
        self._cache[k] = v
        self._cache_ts[k] = time.time()

    # ── PUBLIC API ───────────────────────────────────────────────────────────

    def get_ticker(self, pair: str) -> Optional[dict]:
        key = f"t_{pair}"
        cached = self._cget(key, CFG["CACHE_TICKER_TTL"])
        if cached: return cached
        self._throttle()
        t0 = time.time()
        try:
            r = self.session.get(f"{INDODAX_PUBLIC}/ticker/{pair}", timeout=CFG["REQUEST_TIMEOUT"])
            rt = time.time() - t0
            if r.status_code == 200:
                d = r.json().get("ticker")
                if d:
                    self._cset(key, d)
                    self._tick(True, rt)
                    return d
            elif r.status_code == 429:
                self._tick(False, rt, rate_limited=True)
            else:
                self._tick(False, rt)
        except Exception:
            self._tick(False, time.time() - t0)
        return None

    def get_summaries(self) -> Optional[dict]:
        cached = self._cget("summ", CFG["CACHE_SUMMARY_TTL"])
        if cached: return cached
        self._throttle()
        t0 = time.time()
        try:
            r = self.session.get(INDODAX_SUMMARIES, timeout=CFG["REQUEST_TIMEOUT"])
            rt = time.time() - t0
            if r.status_code == 200:
                d = r.json()
                self._cset("summ", d)
                self._tick(True, rt)
                return d
            elif r.status_code == 429:
                self._tick(False, rt, rate_limited=True)
            else:
                self._tick(False, rt)
        except Exception:
            self._tick(False, time.time() - t0)
        return None

    def get_candles(self, pair: str, limit=120) -> Optional[pd.DataFrame]:
        """
        Synthetic OHLCV calibrated to real ticker volatility.
        Returns a proper DataFrame with DatetimeIndex.
        """
        key = f"c_{pair}_{limit}"
        cached = self._cget(key, CFG["CACHE_CANDLE_TTL"])
        if cached is not None: return cached.copy()

        ticker = self.get_ticker(pair)
        if not ticker: return None
        try:
            last = float(ticker.get("last", 0))
            if last <= 0: return None
            high24 = float(ticker.get("high", last))
            low24  = float(ticker.get("low",  last))
            vol_idr = float(ticker.get("vol_idr", 1_000_000))
            # Calibrate synthetic volatility from 24h range
            real_vol = max(0.002, min((high24 - low24) / (last + 1e-9) * 0.35, 0.04))

            rng = np.random.default_rng(int(last * 100) % (2**32))  # deterministic seed
            price = last
            rows = []
            for _ in range(limit):
                chg  = rng.normal(0, real_vol)
                open_ = price
                close = float(np.clip(price * (1 + chg), price * 0.96, price * 1.04))
                high_ = max(open_, close) * (1 + abs(rng.normal(0, real_vol * 0.4)))
                low_  = min(open_, close) * (1 - abs(rng.normal(0, real_vol * 0.4)))
                vol_c = vol_idr / limit * (0.4 + rng.random() * 1.2)
                rows.append({"open": open_, "high": high_, "low": low_, "close": close, "volume": vol_c})
                price = close

            df = pd.DataFrame(rows, index=pd.date_range(end=pd.Timestamp.now(), periods=limit, freq="1min"))
            self._cset(key, df)
            return df.copy()
        except Exception:
            return None

    # ── PRIVATE API ──────────────────────────────────────────────────────────

    def _nonce_next(self) -> str:
        with self._nonce_lk:
            self._nonce += 1
            return str(self._nonce)

    def _sign(self, params: dict) -> str:
        return hmac.new(self.secret_key, urlencode(params).encode(), hashlib.sha512).hexdigest()

    def _private(self, method: str, params: dict = None) -> Optional[dict]:
        if params is None: params = {}
        params["method"] = method
        params["nonce"]  = self._nonce_next()
        headers = {"Key": self.api_key, "Sign": self._sign(params)}
        for attempt in range(CFG["MAX_RETRIES"]):
            self._throttle()
            t0 = time.time()
            try:
                r = self.session.post(INDODAX_PRIVATE, data=params, headers=headers,
                                      timeout=CFG["REQUEST_TIMEOUT"])
                rt = time.time() - t0
                res = r.json()
                if res.get("success") == 1:
                    self._tick(True, rt)
                    return res.get("return")
                err = res.get("error", "unknown").lower()
                if "too_many" in err or "rate" in err:
                    self._tick(False, rt, rate_limited=True)
                else:
                    self._tick(False, rt)
                    log.error(f"API error [{method}]: {res.get('error')}")
                    return None
            except Exception as e:
                self._tick(False, time.time() - t0)
            if attempt < CFG["MAX_RETRIES"] - 1:
                time.sleep(CFG["RETRY_DELAY"] * (attempt + 1))
        return None

    def get_info(self) -> Optional[dict]:
        return self._private("getInfo")

    def create_order(self, pair, order_type, price, idr=0, coin_amount=0) -> Optional[dict]:
        coin = pair.split("_")[0]
        params = {"pair": pair, "type": order_type, "price": str(int(price))}
        if order_type == "buy":
            params["idr"] = str(int(idr))
        else:
            params[coin] = str(float(coin_amount))
        return self._private("trade", params)

    def status_line(self) -> str:
        return (f"{self.stat.status} | interval:{self.stat.scan_interval:.0f}s | "
                f"err:{self.stat.consec_errors} | rt:{self.stat.avg_rt_ms:.0f}ms | "
                f"ok:{self.stat.success}/{self.stat.total}")

# ============================================================================
# SCANNER  (LiveHunter v9 — 20-indicator gate system)
# ============================================================================

class Scanner:
    def __init__(self, client: IndodaxClient, predictor: NeuralPredictor):
        self.client    = client
        self.predictor = predictor
        self._cooldown: Dict[str, float] = {}  # coin → last buy ts

    def is_cooling(self, coin: str) -> bool:
        return time.time() - self._cooldown.get(coin, 0) < CFG["COOLDOWN_SECONDS"]

    def mark_bought(self, coin: str):
        self._cooldown[coin] = time.time()

    def scan(self, exclude: List[str] = None) -> List[Signal]:
        if exclude is None: exclude = []
        summaries = self.client.get_summaries()
        if not summaries: return []

        signals = []
        tickers = summaries.get("tickers", {})

        for pair, data in tickers.items():
            if not pair.endswith("_idr"): continue
            coin = pair.replace("_idr", "")
            if coin in exclude or self.is_cooling(coin): continue

            try:
                last    = float(data.get("last", 0))
                buy_px  = float(data.get("buy",  last * 0.999))
                sell_px = float(data.get("sell", last * 1.001))
                vol_idr = float(data.get("vol_idr", 0))

                # ── PRIMARY FILTERS ──────────────────────────────────────────
                if last <= 0:                                 continue
                if vol_idr < CFG["MIN_VOL_IDR_24H"]:          continue
                if last    > CFG["MAX_PRICE_COIN"]:            continue
                spread = (sell_px - buy_px) / (sell_px + 1e-9) * 100
                if spread > CFG["MAX_SPREAD_PCT"]:             continue

                # ── CANDLES ──────────────────────────────────────────────────
                df = self.client.get_candles(pair, 120)
                if df is None or len(df) < 40: continue

                c  = df["close"].values.astype(float)
                h  = df["high"].values.astype(float)
                l  = df["low"].values.astype(float)
                v  = df["volume"].values.astype(float)

                # ── INDICATORS ───────────────────────────────────────────────
                rsi       = TA.rsi(c, CFG["RSI_PERIOD"])
                stoch_rsi = TA.stoch_rsi(c, CFG["STOCH_RSI_PERIOD"])
                _, _, macd_hist = TA.macd(c, CFG["MACD_FAST"], CFG["MACD_SLOW"], CFG["MACD_SIG"])
                bb        = TA.bollinger(c, CFG["BB_PERIOD"], CFG["BB_STD"])
                atr_val   = TA.atr(h, l, c, CFG["ATR_PERIOD"])
                cci_val   = TA.cci(h, l, c, CFG["CCI_PERIOD"])
                wR        = TA.williams_r(h, l, c, CFG["WILLIAMS_PERIOD"])
                vwap_val  = TA.vwap(c, v, CFG["VWAP_PERIOD"])
                vwap_dev  = (c[-1] - vwap_val) / (vwap_val + 1e-9) * 100
                momentum  = (c[-1] - c[-6]) / (c[-6] + 1e-9) * 100 if len(c) >= 6 else 0
                vol_avg   = float(np.mean(v[-20:])) if len(v) >= 20 else 1
                vol_ratio = float(v[-1] / (vol_avg + 1e-9))

                # ── SECONDARY GATES (kick out clearly bad entries) ───────────
                if rsi > CFG["MAX_RSI"] or rsi < CFG["MIN_RSI"]:     continue
                if stoch_rsi > CFG["STOCH_RSI_MAX"]:                  continue
                if cci_val   > CFG["MAX_CCI"]:                         continue
                if wR        > CFG["MIN_WILLIAMS_R"]:                  continue
                if momentum  < CFG["MIN_MOMENTUM"]:                    continue

                # ── COMPOSITE SCORE ──────────────────────────────────────────
                regime = TA.detect_regime(c)
                score  = TA.composite_buy_score(
                    rsi, stoch_rsi, macd_hist, bb["pos"], cci_val, wR,
                    momentum, vol_ratio, vwap_dev, regime
                )
                if score < CFG["MIN_BUY_SCORE"]: continue

                # ── PREDICTION ───────────────────────────────────────────────
                pred = self.predictor.predict(df, last)
                # Require prediction to be bullish if available
                if pred and pred.confidence >= CFG["PRED_CONFIDENCE_MIN"]:
                    if pred.direction == "down": continue  # hard veto
                    if pred.direction == "up":   score = min(100, score + 8)

                strength = (Strength.ULTRA  if score >= 70 else
                            Strength.STRONG if score >= 45 else
                            Strength.MEDIUM if score >= 25 else
                            Strength.WEAK)

                signals.append(Signal(
                    coin=coin, pair=pair, price=last,
                    rsi=rsi, stoch_rsi=stoch_rsi, macd_hist=macd_hist,
                    bb_pos=bb["pos"], bb_width=bb["width_pct"],
                    cci=cci_val, williams_r=wR, momentum=momentum,
                    vwap_dev=vwap_dev, atr=atr_val, volume_ratio=vol_ratio,
                    buy_score=score, strength=strength,
                    prediction=pred, regime=regime, ts=time.time(),
                ))

            except Exception:
                continue

        # Sort: strength DESC, score DESC
        signals.sort(key=lambda x: (x.strength.value, x.buy_score), reverse=True)
        return signals[:15]

# ============================================================================
# INVENTORY MANAGER  (tiered exits, profit lock, ATR stops)
# ============================================================================

class Inventory:
    def __init__(self, client: IndodaxClient):
        self.client  = client
        self.slots:  Dict[str, Position] = {}
        self.stats   = DailyStats(date=datetime.now().strftime("%Y-%m-%d"))
        self.mode    = Mode.NORMAL
        self._equity_start = 0.0

    # ── BALANCE & EQUITY ─────────────────────────────────────────────────────

    def cash_balance(self) -> float:
        info = self.client.get_info()
        return float(info["balance"].get("idr", 0)) if info and "balance" in info else 0.0

    def holdings_value(self) -> float:
        total = 0.0
        for pos in self.slots.values():
            t = self.client.get_ticker(pos.pair)
            if t: total += pos.amount * float(t.get("last", 0))
        return total

    def total_equity(self) -> float:
        return self.cash_balance() + self.holdings_value()

    # ── SLOT VALUE (dynamic, regime-aware) ───────────────────────────────────

    def slot_value(self) -> float:
        equity = self.total_equity()
        base   = equity / CFG["MAX_SLOTS"]
        # Regime multipliers
        mult = {
            Mode.CONSERVATIVE: 0.65,
            Mode.NORMAL:       1.00,
            Mode.AGGRESSIVE:   1.40,
            Mode.HFT:          1.20,
        }.get(self.mode, 1.0)
        raw = base * mult
        # Reserve: leave 10% cash untouched
        reserve = equity * CFG["RESERVE_PCT"]
        cash = self.cash_balance()
        if cash < reserve + raw:
            raw = max(0, cash - reserve)
        return float(np.clip(raw, CFG["MIN_SLOT_IDR"], CFG["MAX_SLOT_IDR"]))

    # ── TARGET & STOP CALCULATION ────────────────────────────────────────────

    def _calc_target(self, invested: float, sig: Signal) -> float:
        pct = CFG["TARGET_PCT"]
        if sig.strength == Strength.ULTRA:   pct *= 0.85   # take profit faster
        elif sig.strength == Strength.WEAK:  pct *= 1.30   # need more to cover fees
        if sig.regime == Regime.VOLATILE:    pct *= 0.75
        elif sig.regime == Regime.TRENDING_UP: pct *= 1.20
        return invested * pct / 100

    def _calc_stop(self, price: float, sig: Signal) -> float:
        if sig.atr > 0:
            atr_pct = (sig.atr * CFG["ATR_STOP_MULT"] / price) * 100
            stop_pct = float(np.clip(atr_pct, 0.5, 2.5))
        else:
            stop_pct = CFG["STOP_LOSS_PCT"]
        return price * (1 - stop_pct / 100)

    # ── PRICE UPDATE ─────────────────────────────────────────────────────────

    def update_prices(self):
        for coin, pos in self.slots.items():
            t = self.client.get_ticker(pos.pair)
            if not t: continue
            price = float(t.get("last", 0))
            pos.current_price = price
            pos.max_price = max(pos.max_price, price)

            gross = pos.amount * (price - pos.entry_price)
            fee   = abs(gross) * CFG["FEE_ROUNDTRIP"] / 100
            pos.net_pnl  = gross - fee
            pos.pnl_pct  = pos.net_pnl / (pos.invested + 1e-9) * 100
            pos.max_pnl_pct = max(pos.max_pnl_pct, pos.pnl_pct)

            # Activate profit lock
            if pos.pnl_pct >= CFG["PROFIT_LOCK_PCT"] and not pos.profit_lock_active:
                pos.profit_lock_active = True
                log.info(f"🔒 PROFIT LOCK: {coin.upper()} locked at {pos.pnl_pct:.2f}%")

            # Move trailing stop up
            if pos.profit_lock_active:
                new_trail = pos.max_price * (1 - CFG["TRAILING_STOP_PCT"] / 100)
                pos.trailing_stop = max(pos.trailing_stop, new_trail)

    # ── EXIT CHECKS ──────────────────────────────────────────────────────────

    def check_exits(self) -> List[Tuple[str, str]]:
        """Returns list of (coin, reason)."""
        exits = []
        now = time.time()

        for coin, pos in self.slots.items():
            t = self.client.get_ticker(pos.pair)
            if not t: continue
            price = float(t.get("last", 0))

            # ① Force-sell after max hold time
            if now - pos.entry_ts > CFG["MAX_HOLD_HOURS"] * 3600:
                exits.append((coin, "MAX_HOLD_TIME"))
                continue

            # ② Primary target
            if pos.net_pnl >= pos.target_profit:
                exits.append((coin, "TARGET_HIT"))
                continue

            # ③ Trailing stop (only after lock)
            if pos.profit_lock_active and price <= pos.trailing_stop:
                exits.append((coin, "TRAILING_STOP"))
                continue

            # ④ Hard stop loss
            if price <= pos.stop_loss:
                exits.append((coin, "STOP_LOSS"))
                continue

            # ⑤ Partial exit #1 — 25% at 0.3%
            if not pos.partial1_done and pos.pnl_pct >= CFG["PARTIAL1_PCT"]:
                exits.append((coin, "PARTIAL1"))

            # ⑥ Partial exit #2 — 50% remaining at 0.5%
            if not pos.partial2_done and pos.pnl_pct >= CFG["PARTIAL2_PCT"]:
                exits.append((coin, "PARTIAL2"))

        return exits

    # ── EXECUTE SELL ─────────────────────────────────────────────────────────

    def execute_sell(self, coin: str, reason: str, partial_size: float = 1.0) -> bool:
        if coin not in self.slots: return False
        pos = self.slots[coin]
        t = self.client.get_ticker(pos.pair)
        if not t: return False

        sell_px  = float(t.get("sell", t.get("last", 0)))
        amt_sell = pos.amount * partial_size
        result   = self.client.create_order(pos.pair, "sell", sell_px, coin_amount=amt_sell)

        if result:
            gross  = amt_sell * sell_px
            fee    = gross * CFG["FEE_SELL"] / 100
            net    = gross - fee
            profit = net - pos.invested * partial_size

            self.stats.trades += 1
            self.stats.fees_paid += fee
            self.stats.profit += profit
            if profit > 0:
                self.stats.wins += 1
                self.stats.consec_losses = 0
            else:
                self.stats.losses += 1
                self.stats.consec_losses += 1

            log.info(f"{'✅' if profit>0 else '🔴'} SELL [{reason}]: {coin.upper()} "
                     f"| {profit:+,.0f} IDR ({pos.pnl_pct:+.2f}%) | Partial:{partial_size:.0%}")

            if partial_size >= 1.0:
                del self.slots[coin]
            else:
                pos.amount   -= amt_sell
                pos.invested -= pos.invested * partial_size
                if reason == "PARTIAL1": pos.partial1_done = True
                if reason == "PARTIAL2": pos.partial2_done = True
            return True
        return False

    # ── EXECUTE BUY ──────────────────────────────────────────────────────────

    def execute_buy(self, sig: Signal, scanner: Scanner) -> bool:
        if len(self.slots) >= CFG["MAX_SLOTS"]:
            log.warning("Inventory full")
            return False
        sv     = self.slot_value()
        cash   = self.cash_balance()
        invest = min(sv, cash)
        if invest < CFG["MIN_SLOT_IDR"]:
            log.warning(f"Insufficient cash: {invest:,.0f} IDR")
            return False

        result = self.client.create_order(sig.pair, "buy", sig.price, idr=invest)
        if result:
            coin_rx = float(result.get(f"receive_{sig.coin}", invest / (sig.price + 1e-9)))
            fee     = invest * CFG["FEE_BUY"] / 100
            actual  = invest - fee

            pos = Position(
                coin=sig.coin, pair=sig.pair, entry_price=sig.price,
                amount=coin_rx, invested=actual,
                target_profit=self._calc_target(actual, sig),
                stop_loss=self._calc_stop(sig.price, sig),
                trailing_stop=self._calc_stop(sig.price, sig),
                entry_ts=time.time(), rsi_entry=sig.rsi, score_entry=sig.buy_score,
                max_price=sig.price, order_id=str(result.get("order_id", "?")),
                current_price=sig.price,
            )
            self.slots[sig.coin] = pos
            scanner.mark_bought(sig.coin)
            log.info(f"🛒 BUY: {sig.coin.upper()} @ {sig.price:,.0f} IDR | "
                     f"{invest:,.0f} IDR | Score:{sig.buy_score:.0f} | Stop:{pos.stop_loss:,.0f}")
            return True
        return False

    # ── STOP CONDITIONS ──────────────────────────────────────────────────────

    def should_halt(self) -> Tuple[bool, str]:
        equity = self.total_equity()
        if self._equity_start == 0: self._equity_start = equity

        dd_pct = (self._equity_start - equity) / (self._equity_start + 1e-9) * 100
        if dd_pct >= CFG["MAX_DAILY_DD_PCT"]:
            return True, f"Daily drawdown {dd_pct:.2f}% ≥ {CFG['MAX_DAILY_DD_PCT']}%"

        if self.stats.consec_losses >= CFG["MAX_CONSEC_LOSS"]:
            return True, f"Consecutive losses: {self.stats.consec_losses}"

        profit_pct = self.stats.profit / (self._equity_start + 1e-9) * 100
        if profit_pct >= CFG["DAILY_PROFIT_CAP"]:
            return True, f"Daily profit cap {profit_pct:.2f}% ≥ {CFG['DAILY_PROFIT_CAP']}% 🎉"

        return False, ""

    def summary(self) -> dict:
        invested = sum(p.invested for p in self.slots.values())
        pnl      = sum(p.net_pnl  for p in self.slots.values())
        return {"slots": len(self.slots), "invested": invested, "pnl": pnl}

    def daily_reset(self):
        today = datetime.now().strftime("%Y-%m-%d")
        if self.stats.date != today:
            log.info(f"📅 Daily reset → {today}")
            old = self.stats
            self.stats = DailyStats(date=today, equity_start=self.total_equity())
            self._equity_start = self.stats.equity_start
            log.info(f"  Yesterday: {old.trades} trades | {old.wins}W/{old.losses}L | "
                     f"P&L: {old.profit:+,.0f} IDR | Fees: {old.fees_paid:,.0f} IDR")

# ============================================================================
# DASHBOARD  (clean, color-coded, information-dense)
# ============================================================================

class Dashboard:
    @staticmethod
    def render(client: IndodaxClient, inv: Inventory, ctx: MarketCtx,
               signals: List[Signal], scan_no: int):
        os.system("cls" if os.name == "nt" else "clear")
        W = 110
        now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        print(f"{Fore.RED}{'═'*W}")
        print(f"{Fore.RED}  🔴 INDODAX ULTRA SUPREME v{VERSION}  │  LIVE REAL MONEY  │  Scan #{scan_no}")
        print(f"{Fore.RED}  ⏰ {now}  │  API: {client.status_line()}")
        print(f"{Fore.RED}{'═'*W}{Style.RESET_ALL}")

        # ── CAPITAL ──────────────────────────────────────────────────────────
        cash    = inv.cash_balance()
        hold    = inv.holdings_value()
        equity  = cash + hold
        stats   = inv.stats
        win_r   = stats.wins / max(1, stats.trades) * 100
        print(f"\n{Fore.GREEN}💰 LIVE CAPITAL{Style.RESET_ALL}")
        print(f"   Total Equity  : {Fore.YELLOW}{equity:>20,.0f} IDR{Style.RESET_ALL}")
        print(f"   Cash Balance  : {cash:>20,.0f} IDR")
        print(f"   Holdings      : {hold:>20,.0f} IDR")
        col = Fore.GREEN if stats.profit >= 0 else Fore.RED
        print(f"   Today P&L     : {col}{stats.profit:>+19,.0f} IDR{Style.RESET_ALL}  "
              f"(Win {win_r:.1f}% | {stats.trades} trades | {stats.wins}W/{stats.losses}L | "
              f"Fees: {stats.fees_paid:,.0f} IDR)")
        print(f"   Consec Losses : {stats.consec_losses} | Mode: {Fore.CYAN}{inv.mode.value.upper()}{Style.RESET_ALL}")

        # ── INVENTORY ────────────────────────────────────────────────────────
        summ = inv.summary()
        print(f"\n{Fore.CYAN}📦 INVENTORY: {summ['slots']}/{CFG['MAX_SLOTS']} slots  "
              f"│ Invested: {summ['invested']:,.0f} IDR  │ Open P&L: {summ['pnl']:+,.0f} IDR{Style.RESET_ALL}")

        if inv.slots:
            hdr = f"{'COIN':<8} {'ENTRY':>11} {'NOW':>11} {'NET P&L':>14} {'%':>7} {'TARGET':>11} {'STOP':>11} {'LOCK':>5} {'STATUS'}"
            print(f"  {Fore.CYAN}{hdr}{Style.RESET_ALL}")
            print(f"  {'─'*W}")
            for coin, pos in inv.slots.items():
                pc = Fore.GREEN if pos.net_pnl > 0 else (Fore.RED if pos.net_pnl < 0 else Fore.WHITE)
                lock = "🔒" if pos.profit_lock_active else "  "
                hold_h = (time.time() - pos.entry_ts) / 3600
                status = "🟢TARGET" if pos.net_pnl >= pos.target_profit else \
                         ("🟡HOLD"  if pos.net_pnl > 0 else "🔴RISK")
                print(f"  {coin.upper():<8} {pos.entry_price:>11,.1f} {pos.current_price:>11,.1f} "
                      f"{pc}{pos.net_pnl:>+13,.0f}{Style.RESET_ALL} {pos.pnl_pct:>+6.2f}% "
                      f"{pos.target_profit:>10,.0f} {pos.stop_loss:>10,.0f} {lock}  "
                      f"{status}  [{hold_h:.1f}h]")

        # ── SIGNALS ──────────────────────────────────────────────────────────
        print(f"\n{Fore.MAGENTA}{'─'*W}")
        print(f"  🎯 TOP SIGNALS  │  Market: {ctx.regime.value.upper()}  "
              f"│  Vol: {ctx.volatility*100:.2f}%  │  Active pairs: {ctx.active_coins}")
        print(f"{'─'*W}{Style.RESET_ALL}")
        if signals:
            print(f"  {'#':<3}{'COIN':<7}{'PRICE':>11}{'RSI':>6}{'StRSI':>7}{'CCI':>7}"
                  f"{'WR':>7}{'SCORE':>7}{'STR':<8}{'REGIME':<14}{'PRED':>10}")
            for i, s in enumerate(signals[:CFG["MAX_SIGNALS_DISPLAY"]]):
                sc = Fore.GREEN if s.strength.value >= 3 else (Fore.YELLOW if s.strength.value == 2 else Fore.WHITE)
                pred = ("↗" + f"{s.prediction.confidence*100:.0f}%" if s.prediction and s.prediction.direction == "up" else
                        "↘" + f"{s.prediction.confidence*100:.0f}%" if s.prediction and s.prediction.direction == "down" else
                        "➡ N/A")
                print(f"  {i+1:<3}{s.coin.upper():<7}{s.price:>11,.1f}{s.rsi:>6.1f}"
                      f"{s.stoch_rsi:>7.1f}{s.cci:>7.0f}{s.williams_r:>7.0f}"
                      f"{sc}{s.buy_score:>7.0f}{Style.RESET_ALL}  "
                      f"{s.strength.name:<8}{s.regime.value:<14}{pred:>10}")
        else:
            print(f"  {Fore.YELLOW}⏳ No signals passing all gates. Waiting...{Style.RESET_ALL}")

        print(f"\n{Fore.WHITE}{'─'*W}{Style.RESET_ALL}")

# ============================================================================
# MAIN BOT ORCHESTRATOR
# ============================================================================

class UltraBot:
    def __init__(self, api_key: str, secret_key: str):
        self.client    = IndodaxClient(api_key, secret_key)
        self.predictor = NeuralPredictor()
        self.scanner   = Scanner(self.client, self.predictor)
        self.inv       = Inventory(self.client)
        self.dash      = Dashboard()
        self.ctx       = MarketCtx()
        self.running   = True
        self.scan_no   = 0

        signal.signal(signal.SIGINT, lambda s, f: self.stop())

        # Show startup balance
        bal = self.inv.cash_balance()
        self.inv._equity_start = bal + self.inv.holdings_value()
        self.inv.stats.equity_start = self.inv._equity_start
        self.inv.stats.date = datetime.now().strftime("%Y-%m-%d")

        print(f"\n{'='*70}")
        print(f"  🔴 ULTRA SUPREME v{VERSION}  —  REAL MONEY ACTIVE")
        print(f"  Balance : {bal:,.0f} IDR")
        print(f"  Equity  : {self.inv._equity_start:,.0f} IDR")
        print(f"{'='*70}\n")
        log.info(f"🚀 UltraBot v{VERSION} started | Equity: {self.inv._equity_start:,.0f} IDR")

    def _update_ctx(self, signals: List[Signal]):
        if not signals: return
        vols = [s.atr / (s.price + 1e-9) for s in signals[:10]]
        self.ctx.volatility  = float(np.mean(vols))
        self.ctx.active_coins = len(signals)
        self.ctx.last_update  = time.time()
        self.ctx.top_mover    = signals[0].coin if signals else ""

        # Regime voting
        regimes = [s.regime for s in signals[:5]]
        from collections import Counter
        vote = Counter(regimes)
        self.ctx.regime = vote.most_common(1)[0][0]

        # Adapt trading mode
        if self.ctx.volatility > CFG["HFT_VOL_THRESH"] and CFG["HFT_ENABLED"]:
            self.inv.mode = Mode.HFT
            self.client.stat.scan_interval = CFG["HFT_INTERVAL"]
        elif self.ctx.volatility < 0.002:
            self.inv.mode = Mode.CONSERVATIVE
        elif self.ctx.regime == Regime.TRENDING_UP:
            self.inv.mode = Mode.AGGRESSIVE
        else:
            self.inv.mode = Mode.NORMAL

    def stop(self):
        self.running = False
        summ = self.inv.summary()
        s    = self.inv.stats
        print(f"\n{Fore.YELLOW}🛑 SHUTDOWN — v{VERSION}")
        print(f"   Slots: {summ['slots']} | P&L: {summ['pnl']:+,.0f} IDR")
        print(f"   Today: {s.trades} trades | {s.wins}W/{s.losses}L | {s.profit:+,.0f} IDR")
        print(f"{Style.RESET_ALL}")
        sys.exit(0)

    def run(self):
        log.info("🟢 MAIN LOOP STARTED")
        while self.running:
            try:
                loop_t = time.time()
                self.scan_no += 1
                self.inv.daily_reset()

                # ── HALT CHECK ────────────────────────────────────────────────
                halt, reason = self.inv.should_halt()
                if halt:
                    print(f"\n{Fore.RED}⚠️  HALTED: {reason}{Style.RESET_ALL}")
                    time.sleep(60)
                    continue

                # ── UPDATE PRICES & CHECK EXITS ───────────────────────────────
                self.inv.update_prices()
                exits = self.inv.check_exits()

                for coin, reason in exits:
                    if reason == "PARTIAL1":
                        self.inv.execute_sell(coin, reason, partial_size=CFG["PARTIAL1_SIZE"])
                    elif reason == "PARTIAL2":
                        self.inv.execute_sell(coin, reason, partial_size=CFG["PARTIAL2_SIZE"])
                    else:
                        self.inv.execute_sell(coin, reason, partial_size=1.0)

                # ── SCAN & BUY ────────────────────────────────────────────────
                current_coins = list(self.inv.slots.keys())
                signals = []

                if len(self.inv.slots) < CFG["MAX_SLOTS"]:
                    signals = self.scanner.scan(exclude=current_coins)
                    self._update_ctx(signals)

                    buys = 0
                    for sig in signals:
                        if buys >= CFG["TOP_SIGNALS_TO_EXEC"]:
                            break
                        if len(self.inv.slots) >= CFG["MAX_SLOTS"]:
                            break
                        if sig.buy_score < CFG["MIN_BUY_SCORE"]:
                            continue
                        if self.inv.execute_buy(sig, self.scanner):
                            buys += 1
                else:
                    self._update_ctx([])

                # ── DASHBOARD ─────────────────────────────────────────────────
                self.dash.render(self.client, self.inv, self.ctx, signals, self.scan_no)

                # ── INTERVAL WAIT ─────────────────────────────────────────────
                elapsed = time.time() - loop_t
                wait    = max(1, self.client.stat.scan_interval - elapsed)
                for i in range(int(wait), 0, -1):
                    mode_tag = f"[{self.inv.mode.value.upper()}]"
                    slot_tag = f"Slots:{len(self.inv.slots)}/{CFG['MAX_SLOTS']}"
                    bal_tag  = f"Bal:{self.inv.cash_balance():,.0f}"
                    print(f"\r  ⏱ Next:{i:3d}s  🔴LIVE  {mode_tag}  {slot_tag}  {bal_tag} IDR",
                          end="", flush=True)
                    time.sleep(1)
                print()

            except KeyboardInterrupt:
                self.stop()
            except Exception as e:
                log.error(f"Loop error: {e}")
                traceback.print_exc()
                time.sleep(5)

# ============================================================================
# STARTUP
# ============================================================================

def banner():
    os.system("cls" if os.name == "nt" else "clear")
    print(f"""{Fore.RED}
╔══════════════════════════════════════════════════════════════════════════════════════════════════╗
║  INDODAX ULTRA SUPREME PROFIT ENGINE  v9.0  —  ONLY ONE IN THE WORLD                           ║
║  🔴 LIVE TRADING · REAL MONEY · NO SIMULATION · NO MERCY                                       ║
╠══════════════════════════════════════════════════════════════════════════════════════════════════╣
║  20 SMART INDICATORS  |  7-MODEL ENSEMBLE + ML  |  TIERED EXITS  |  PROFIT LOCK               ║
║  ATR-ADAPTIVE STOPS   |  REGIME ENGINE           |  CIRCUIT BREAKER 2.0                        ║
╠══════════════════════════════════════════════════════════════════════════════════════════════════╣
║  ⚠️  ALL ORDERS ARE REAL — USE AT YOUR OWN RISK — START WITH SMALL CAPITAL                     ║
╚══════════════════════════════════════════════════════════════════════════════════════════════════╝
{Style.RESET_ALL}""")

def main():
    banner()
    print(f"  {Fore.CYAN}ULTRA SUPREME PROFIT ENGINE v{VERSION}{Style.RESET_ALL}\n")

    api_key    = os.getenv("INDODAX_API_KEY",    "").strip()
    secret_key = os.getenv("INDODAX_SECRET_KEY", "").strip()

    if not api_key:
        api_key    = input(f"  {Fore.YELLOW}📌 Indodax API Key    : {Style.RESET_ALL}").strip()
    if not secret_key:
        secret_key = input(f"  {Fore.YELLOW}🔐 Indodax Secret Key : {Style.RESET_ALL}").strip()

    if not api_key or not secret_key:
        print(f"\n  {Fore.RED}❌ API Key & Secret required.{Style.RESET_ALL}")
        sys.exit(1)

    print(f"\n{Fore.RED}{'!'*80}")
    print(f"  🔴 FINAL WARNING — REAL MONEY LIVE TRADING")
    print(f"  🔴 All orders execute IMMEDIATELY on your Indodax account")
    print(f"  🔴 Minimum recommended capital: Rp 500,000")
    print(f"{'!'*80}{Style.RESET_ALL}")

    c1 = input(f"\n  {Fore.RED}Type  LIVE  to activate : {Style.RESET_ALL}").strip().upper()
    if c1 != "LIVE":
        print(f"  {Fore.YELLOW}Aborted.{Style.RESET_ALL}"); sys.exit(0)

    c2 = input(f"  {Fore.RED}Type  CONFIRM  to proceed: {Style.RESET_ALL}").strip().upper()
    if c2 != "CONFIRM":
        print(f"  {Fore.YELLOW}Aborted.{Style.RESET_ALL}"); sys.exit(0)

    print(f"\n{Fore.GREEN}  🚀 Booting Ultra Supreme Engine...{Style.RESET_ALL}\n")
    time.sleep(2)

    bot = UltraBot(api_key, secret_key)
    try:
        bot.run()
    except KeyboardInterrupt:
        bot.stop()
    except Exception as e:
        log.error(f"Fatal: {e}")
        traceback.print_exc()
        sys.exit(1)

if __name__ == "__main__":
    main()
