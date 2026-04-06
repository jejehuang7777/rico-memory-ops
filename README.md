//@version=6
indicator("Room12 Bias / Trigger Session v2", overlay=true, max_labels_count=300, max_lines_count=300, max_boxes_count=50)

// -----------------------------------------------------------------------------
// Room12 custom intraday helper
// Core idea:
// 1) Session reset. No dragging yesterday's entry into today.
// 2) Bias vs Trigger are separated.
// 3) Higher-TF bias, lower-TF pullback confirmation.
// 4) Chop = no-trade / watch-only.
// 5) Flip bias only after 2-3 higher-TF closes beyond the key line.
// -----------------------------------------------------------------------------

// === INPUTS ===
groupSession = "Session"
tradeSession = input.session("0930-1600", "Trade session (exchange time)", group=groupSession)
biasTF       = input.timeframe("3", "Bias timeframe", group=groupSession)
waitAfterOpenMin = input.int(3, "No new entries for first N minutes", minval=0, group=groupSession)

groupBias = "Bias"
fastLen    = input.int(9, "Fast EMA", minval=1, group=groupBias)
slowLen    = input.int(21, "Slow EMA", minval=2, group=groupBias)
adxLen     = input.int(14, "ADX Length", minval=1, group=groupBias)
biasAdxMin = input.float(18.0, "Bias min ADX", minval=0.0, step=0.5, group=groupBias)
flipBars   = input.int(2, "Flip after N bias bars", minval=2, maxval=3, group=groupBias)

groupEntry = "Trigger"
touchMode         = input.string("Ribbon", "Pullback touch zone", options=["EMA9", "EMA21", "Ribbon"], group=groupEntry)
pullbackExpireBars = input.int(6, "Pullback expires after N bars", minval=1, group=groupEntry)
requireBreakPrev  = input.bool(true, "Confirmation must break prior bar", group=groupEntry)
requireVWAPConfirm = input.bool(true, "Long > VWAP / Short < VWAP on trigger", group=groupEntry)
requireCloseTrend = input.bool(true, "Long close > EMA9 / Short close < EMA9", group=groupEntry)

groupChop = "Chop Filter"
chopLookback   = input.int(8, "Cross-count lookback", minval=3, group=groupChop)
chopCrossLimit = input.int(3, "Cross-count no-trade threshold", minval=1, group=groupChop)
chopAdxMin     = input.float(16.0, "Current TF ADX minimum", minval=0.0, step=0.5, group=groupChop)
ribbonAtrFloor = input.float(0.15, "Min EMA ribbon width as ATR fraction", minval=0.01, step=0.01, group=groupChop)

groupRisk = "Risk & Targets"
atrLen    = input.int(14, "ATR Length", minval=1, group=groupRisk)
slAtrFloor = input.float(1.0, "Minimum stop = ATR x", minval=0.1, step=0.1, group=groupRisk)
tp1R      = input.float(1.0, "TP1 = R x", minval=0.25, step=0.25, group=groupRisk)
tp2R      = input.float(2.0, "TP2 = R x", minval=0.25, step=0.25, group=groupRisk)
tp3R      = input.float(3.0, "TP3 = R x", minval=0.25, step=0.25, group=groupRisk)
moveToBEOnTP1 = input.bool(false, "Move SL to breakeven after TP1", group=groupRisk)

groupVisual = "Visuals"
showEMAs      = input.bool(true, "Show EMA 9 / 21", group=groupVisual)
showVWAP      = input.bool(true, "Show chart VWAP", group=groupVisual)
showBiasBg    = input.bool(true, "Tint chart by state", group=groupVisual)
showKeyLine   = input.bool(true, "Show bias key line (HTF VWAP)", group=groupVisual)
showStatusTag = input.bool(true, "Show status tag", group=groupVisual)
showWatchBars = input.bool(true, "Color watch bars", group=groupVisual)
labelOffset   = input.int(12, "Label right offset", minval=1, group=groupVisual)

// === SAFETY CHECK ===
int chartSec = timeframe.in_seconds()
int biasSec  = timeframe.in_seconds(biasTF)
if chartSec > biasSec
    runtime.error("Use a chart timeframe <= bias timeframe. Example: run on 1m chart with 3m bias.")

// === CORE CALCULATIONS (CURRENT CHART TF) ===
emaFast = ta.ema(close, fastLen)
emaSlow = ta.ema(close, slowLen)
vwapVal = ta.vwap(hlc3)
atrVal  = ta.atr(atrLen)
[_, _, adxNow] = ta.dmi(adxLen, adxLen)

// === HTF HELPERS ===
f_htfFast() =>
    ta.ema(close, fastLen)

f_htfSlow() =>
    ta.ema(close, slowLen)

f_htfVWAP() =>
    ta.vwap(hlc3)

f_htfADX() =>
    [_, _, _adx] = ta.dmi(adxLen, adxLen)
    _adx

f_rawBiasState() =>
    _fast = ta.ema(close, fastLen)
    _slow = ta.ema(close, slowLen)
    _vwap = ta.vwap(hlc3)
    [_, _, _adx] = ta.dmi(adxLen, adxLen)
    _bull = close > _vwap and _fast > _slow and _fast > _fast[1] and _adx >= biasAdxMin
    _bear = close < _vwap and _fast < _slow and _fast < _fast[1] and _adx >= biasAdxMin
    _bull ? 1 : _bear ? -1 : 0

f_flipDownReady() =>
    _fast = ta.ema(close, fastLen)
    _slow = ta.ema(close, slowLen)
    _vwap = ta.vwap(hlc3)
    ta.sum((close < _vwap and _fast < _slow ? 1.0 : 0.0), flipBars) == flipBars

f_flipUpReady() =>
    _fast = ta.ema(close, fastLen)
    _slow = ta.ema(close, slowLen)
    _vwap = ta.vwap(hlc3)
    ta.sum((close > _vwap and _fast > _slow ? 1.0 : 0.0), flipBars) == flipBars

htfFast    = request.security(syminfo.tickerid, biasTF, f_htfFast(), lookahead=barmerge.lookahead_off)
htfSlow    = request.security(syminfo.tickerid, biasTF, f_htfSlow(), lookahead=barmerge.lookahead_off)
htfVWAP    = request.security(syminfo.tickerid, biasTF, f_htfVWAP(), lookahead=barmerge.lookahead_off)
htfADX     = request.security(syminfo.tickerid, biasTF, f_htfADX(), lookahead=barmerge.lookahead_off)
rawBias    = request.security(syminfo.tickerid, biasTF, f_rawBiasState(), lookahead=barmerge.lookahead_off)
flipDown   = request.security(syminfo.tickerid, biasTF, f_flipDownReady(), lookahead=barmerge.lookahead_off)
flipUp     = request.security(syminfo.tickerid, biasTF, f_flipUpReady(), lookahead=barmerge.lookahead_off)

// === SESSION CONTROL ===
inSession = not na(time(timeframe.period, tradeSession))
prevInSession = bar_index > 0 ? inSession[1] : false
newSession = inSession and not prevInSession

var int sessionOpenTs = na
if newSession
    sessionOpenTs := time

float minutesFromOpen = na(sessionOpenTs) ? na : (time - sessionOpenTs) / 60000.0
readyAfterOpen = inSession and (na(minutesFromOpen) ? false : minutesFromOpen >= waitAfterOpenMin)

// === REGIME / ONE FLIP PER SESSION ===
var int regime = 0      // 1 bull, -1 bear, 0 wait
var int flipsUsed = 0

if newSession
    regime := 0
    flipsUsed := 0

if inSession and readyAfterOpen
    if regime == 0
        if rawBias == 1
            regime := 1
        else if rawBias == -1
            regime := -1
    else if regime == 1 and flipDown and flipsUsed < 1
        regime := -1
        flipsUsed += 1
    else if regime == -1 and flipUp and flipsUsed < 1
        regime := 1
        flipsUsed += 1

bullBias = inSession and readyAfterOpen and regime == 1
bearBias = inSession and readyAfterOpen and regime == -1

// === CHOP FILTER ===
float crossCount = ta.sum(ta.cross(close, vwapVal) ? 1.0 : 0.0, chopLookback) + ta.sum(ta.cross(close, emaFast) ? 1.0 : 0.0, chopLookback)
float ribbonWidth = math.abs(emaFast - emaSlow)
chop = inSession and (adxNow < chopAdxMin or crossCount >= chopCrossLimit or ribbonWidth < atrVal * ribbonAtrFloor)

// === PULLBACK WATCH STATE ===
var bool watchLong = false
var bool watchShort = false
var int watchLongBar = na
var int watchShortBar = na
var float pullbackLow = na
var float pullbackHigh = na

// === ACTIVE SETUP STATE ===
var int tradeDir = 0
var float entryP = na
var float slP = na
var float tp1 = na
var float tp2 = na
var float tp3 = na
var bool tp1Hit = false
var bool tp2Hit = false
var bool tp3Hit = false
var bool slHit = false

// === DRAWING HANDLES ===
var line lineEntry = na
var line lineSL = na
var line lineTP1 = na
var line lineTP2 = na
var line lineTP3 = na
var label labelEntry = na
var label labelSL = na
var label labelTP1 = na
var label labelTP2 = na
var label labelTP3 = na
var label statusLabel = na

f_clearDrawings() =>
    if not na(lineEntry)
        line.delete(lineEntry)
    if not na(lineSL)
        line.delete(lineSL)
    if not na(lineTP1)
        line.delete(lineTP1)
    if not na(lineTP2)
        line.delete(lineTP2)
    if not na(lineTP3)
        line.delete(lineTP3)
    if not na(labelEntry)
        label.delete(labelEntry)
    if not na(labelSL)
        label.delete(labelSL)
    if not na(labelTP1)
        label.delete(labelTP1)
    if not na(labelTP2)
        label.delete(labelTP2)
    if not na(labelTP3)
        label.delete(labelTP3)

if newSession
    watchLong := false
    watchShort := false
    watchLongBar := na
    watchShortBar := na
    pullbackLow := na
    pullbackHigh := na
    tradeDir := 0
    entryP := na
    slP := na
    tp1 := na
    tp2 := na
    tp3 := na
    tp1Hit := false
    tp2Hit := false
    tp3Hit := false
    slHit := false
    f_clearDrawings()
    lineEntry := na
    lineSL := na
    lineTP1 := na
    lineTP2 := na
    lineTP3 := na
    labelEntry := na
    labelSL := na
    labelTP1 := na
    labelTP2 := na
    labelTP3 := na

// === TOUCH ZONE LOGIC ===
longTouch = touchMode == "EMA9"  ? low <= emaFast :
            touchMode == "EMA21" ? low <= emaSlow :
            (low <= emaFast and high >= emaSlow)

shortTouch = touchMode == "EMA9"  ? high >= emaFast :
             touchMode == "EMA21" ? high >= emaSlow :
             (high >= emaFast and low <= emaSlow)

// Start watch only when the setup first touches the pullback zone.
if bullBias and not chop and not watchLong and longTouch
    watchLong := true
    watchLongBar := bar_index
    pullbackLow := low

if bearBias and not chop and not watchShort and shortTouch
    watchShort := true
    watchShortBar := bar_index
    pullbackHigh := high

// Maintain pullback extremes while the watch is active.
if watchLong
    pullbackLow := na(pullbackLow) ? low : math.min(pullbackLow, low)
if watchShort
    pullbackHigh := na(pullbackHigh) ? high : math.max(pullbackHigh, high)

// Expire stale watches.
if watchLong and (bar_index - watchLongBar > pullbackExpireBars or not bullBias or chop)
    watchLong := false
    watchLongBar := na
    pullbackLow := na

if watchShort and (bar_index - watchShortBar > pullbackExpireBars or not bearBias or chop)
    watchShort := false
    watchShortBar := na
    pullbackHigh := na

// === CONFIRMATION / TRIGGER ===
longCore  = close > open and close >= close[1] and (not requireCloseTrend or close > emaFast) and (not requireVWAPConfirm or close > vwapVal)
shortCore = close < open and close <= close[1] and (not requireCloseTrend or close < emaFast) and (not requireVWAPConfirm or close < vwapVal)
longBreak = high > high[1]
shortBreak = low < low[1]

longTrigger = barstate.isconfirmed and watchLong and bullBias and not chop and longCore and (not requireBreakPrev or longBreak)
shortTrigger = barstate.isconfirmed and watchShort and bearBias and not chop and shortCore and (not requireBreakPrev or shortBreak)

// === CREATE NEW SETUP ===
if longTrigger or shortTrigger
    f_clearDrawings()
    lineEntry := na
    lineSL := na
    lineTP1 := na
    lineTP2 := na
    lineTP3 := na
    labelEntry := na
    labelSL := na
    labelTP1 := na
    labelTP2 := na
    labelTP3 := na

    tradeDir := longTrigger ? 1 : -1
    entryP := close

    float structureRisk = longTrigger ? (entryP - nz(pullbackLow, low)) : (nz(pullbackHigh, high) - entryP)
    float atrRisk = atrVal * slAtrFloor
    float finalRisk = math.max(atrRisk, math.max(structureRisk, syminfo.mintick * 4))

    slP := longTrigger ? entryP - finalRisk : entryP + finalRisk
    tp1 := longTrigger ? entryP + finalRisk * tp1R : entryP - finalRisk * tp1R
    tp2 := longTrigger ? entryP + finalRisk * tp2R : entryP - finalRisk * tp2R
    tp3 := longTrigger ? entryP + finalRisk * tp3R : entryP - finalRisk * tp3R

    tp1Hit := false
    tp2Hit := false
    tp3Hit := false
    slHit := false

    lineEntry := line.new(bar_index, entryP, bar_index + 30, entryP, extend=extend.right, color=color.blue, width=2)
    lineSL    := line.new(bar_index, slP,    bar_index + 30, slP,    extend=extend.right, color=color.red, width=2)
    lineTP1   := line.new(bar_index, tp1,    bar_index + 30, tp1,    extend=extend.right, color=color.green, style=line.style_dashed)
    lineTP2   := line.new(bar_index, tp2,    bar_index + 30, tp2,    extend=extend.right, color=color.green, style=line.style_dashed)
    lineTP3   := line.new(bar_index, tp3,    bar_index + 30, tp3,    extend=extend.right, color=color.rgb(0, 90, 0), width=2)

    labelEntry := label.new(bar_index + labelOffset, entryP, "ENTRY: " + str.tostring(entryP, "#.##"), style=label.style_label_left, color=color.blue, textcolor=color.white, size=size.small)
    labelSL    := label.new(bar_index + labelOffset, slP,    "SL: "    + str.tostring(slP, "#.##"),    style=label.style_label_left, color=color.red, textcolor=color.white, size=size.small)
    labelTP1   := label.new(bar_index + labelOffset, tp1,    "TP1: "   + str.tostring(tp1, "#.##"),    style=label.style_label_left, color=color.green, textcolor=color.white, size=size.small)
    labelTP2   := label.new(bar_index + labelOffset, tp2,    "TP2: "   + str.tostring(tp2, "#.##"),    style=label.style_label_left, color=color.green, textcolor=color.white, size=size.small)
    labelTP3   := label.new(bar_index + labelOffset, tp3,    "TP3: "   + str.tostring(tp3, "#.##"),    style=label.style_label_left, color=color.rgb(0, 90, 0), textcolor=color.white, size=size.small)

    watchLong := false
    watchShort := false
    watchLongBar := na
    watchShortBar := na
    pullbackLow := na
    pullbackHigh := na

// === ACTIVE SETUP TRACKING ===
if tradeDir == 1 and not na(entryP) and not slHit
    tp1Hit := tp1Hit or high >= tp1
    tp2Hit := tp2Hit or high >= tp2
    tp3Hit := tp3Hit or high >= tp3
    slHit  := slHit or low <= slP

if tradeDir == -1 and not na(entryP) and not slHit
    tp1Hit := tp1Hit or low <= tp1
    tp2Hit := tp2Hit or low <= tp2
    tp3Hit := tp3Hit or low <= tp3
    slHit  := slHit or high >= slP

if moveToBEOnTP1 and not na(entryP) and tp1Hit and not slHit
    if tradeDir == 1 and slP < entryP
        slP := entryP
    if tradeDir == -1 and slP > entryP
        slP := entryP

// Update line styling after TP / SL events.
if not na(lineSL)
    line.set_y1(lineSL, slP)
    line.set_y2(lineSL, slP)
    line.set_color(lineSL, slHit ? color.gray : color.red)

if not na(lineTP1)
    line.set_color(lineTP1, tp1Hit ? color.aqua : color.green)
if not na(lineTP2)
    line.set_color(lineTP2, tp2Hit ? color.aqua : color.green)
if not na(lineTP3)
    line.set_color(lineTP3, tp3Hit ? color.aqua : color.rgb(0, 90, 0))

if barstate.islast and not na(entryP) and not na(labelEntry)
    int x = bar_index + labelOffset
    label.set_xy(labelEntry, x, entryP)
    label.set_text(labelEntry, "ENTRY: " + str.tostring(entryP, "#.##"))
    label.set_xy(labelSL, x, slP)
    label.set_text(labelSL, "SL: " + str.tostring(slP, "#.##") + (slHit ? " ✕" : ""))
    label.set_xy(labelTP1, x, tp1)
    label.set_text(labelTP1, "TP1: " + str.tostring(tp1, "#.##") + (tp1Hit ? " ✓" : ""))
    label.set_xy(labelTP2, x, tp2)
    label.set_text(labelTP2, "TP2: " + str.tostring(tp2, "#.##") + (tp2Hit ? " ✓" : ""))
    label.set_xy(labelTP3, x, tp3)
    label.set_text(labelTP3, "TP3: " + str.tostring(tp3, "#.##") + (tp3Hit ? " ✓" : ""))

// === VISUALS ===
stateBg = not inSession ? na :
          chop ? color.new(color.gray, 88) :
          bullBias ? color.new(color.green, 92) :
          bearBias ? color.new(color.red, 92) : color.new(color.gray, 94)

bgcolor(showBiasBg ? stateBg : na)

plot(showEMAs ? emaFast : na, "EMA Fast", color=color.new(color.green, 0), linewidth=2)
plot(showEMAs ? emaSlow : na, "EMA Slow", color=color.new(color.red, 0), linewidth=2)
plot(showVWAP ? vwapVal : na, "Chart VWAP", color=color.new(color.orange, 0), linewidth=2)
plot(showKeyLine ? htfVWAP : na, "Bias Key Line (HTF VWAP)", color=color.new(color.blue, 20), linewidth=2, style=plot.style_linebr)

plotshape(longTrigger, title="Room12 Long Trigger", style=shape.labelup, location=location.belowbar, color=color.green, textcolor=color.white, text="BUY", size=size.small)
plotshape(shortTrigger, title="Room12 Short Trigger", style=shape.labeldown, location=location.abovebar, color=color.red, textcolor=color.white, text="SELL", size=size.small)
barcolor(showWatchBars ? (watchLong ? color.orange : watchShort ? color.purple : na) : na)

// === STATUS TAG ===
if barstate.islast and showStatusTag
    if not na(statusLabel)
        label.delete(statusLabel)

    string lightText = not inSession ? "⛔ SESSION OFF" :
         not readyAfterOpen ? "⛔ OPENING LOCK" :
         chop ? "⛔ CHOP / NO TRADE" :
         longTrigger ? "🟢 LONG READY" :
         shortTrigger ? "🟢 SHORT READY" :
         bullBias ? "⚠️ BULL BIAS / WAIT" :
         bearBias ? "⚠️ BEAR BIAS / WAIT" : "⛔ NO BIAS"

    string biasText = regime == 1 ? "Bias: BULL" : regime == -1 ? "Bias: BEAR" : "Bias: WAIT"
    string flipText = regime == 1 ? (flipsUsed < 1 ? "Flip->Short: need " + str.tostring(flipBars) + " x " + biasTF + " closes below key" : "Flip used this session") :
                      regime == -1 ? (flipsUsed < 1 ? "Flip->Long: need " + str.tostring(flipBars) + " x " + biasTF + " closes above key" : "Flip used this session") :
                      "Waiting for first regime"
    string chopText = chop ? "Chop: YES" : "Chop: NO"

    color statusColor = chop ? color.gray : longTrigger ? color.green : shortTrigger ? color.red : bullBias ? color.green : bearBias ? color.red : color.gray
    float y = high + atrVal * 1.2
    statusLabel := label.new(bar_index + 2, y, lightText + "\n" + biasText + "\n" + chopText + "\n" + flipText, style=label.style_label_upper_left, color=color.new(color.black, 75), textcolor=statusColor, size=size.small)

// === ALERTS ===
alertcondition(longTrigger, title="Room12 Long Trigger", message="Room12 long trigger on {{ticker}} @ {{close}}")
alertcondition(shortTrigger, title="Room12 Short Trigger", message="Room12 short trigger on {{ticker}} @ {{close}}")
alertcondition(chop, title="Room12 Chop", message="Room12 chop / no-trade zone on {{ticker}}")
