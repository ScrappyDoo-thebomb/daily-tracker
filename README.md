const STORAGE_KEY = 'daily-button-tracker-v1';

const currentDateKey = () => new Date().toISOString().slice(0, 10);

const safeNumber = (value) => {
  const parsed = Number(value);
  return Number.isFinite(parsed) ? parsed : 0;
};

const loadData = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) return {};
    const parsed = JSON.parse(saved);
    return parsed && typeof parsed === 'object' ? parsed : {};
  } catch (error) {
    console.warn('Unable to read tracker data from localStorage.', error);
    return {};
  }
};

const saveData = (data) => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
};

const ensureTodayEntry = (data) => {
  const today = currentDateKey();
  if (!data[today]) {
    data[today] = { date: today, correct: 0, wrong: 0 };
  }
  return data[today];
};

const formatDateLabel = (dateString) => {
  const date = new Date(`${dateString}T00:00:00`);
  return new Intl.DateTimeFormat('en-US', { month: 'short', day: 'numeric' }).format(date);
};

const calculatePercentageChange = (start, end) => {
  if (!start && !end) return 0;
  if (!start && end > 0) return 100;
  return Number((((end - start) / start) * 100).toFixed(1));
};

const getTrendSummary = (series) => {
  const firstValue = series[0] ?? 0;
  const lastValue = series[series.length - 1] ?? 0;
  return {
    firstValue,
    lastValue,
    percentage: calculatePercentageChange(firstValue, lastValue)
  };
};

const getTodaySummary = (data, dateKey) => {
  const entry = data[dateKey] || { correct: 0, wrong: 0 };
  return {
    correct: safeNumber(entry.correct),
    wrong: safeNumber(entry.wrong)
  };
};

const setButtonPressedState = (button) => {
  button.classList.add('is-pressed');
  clearTimeout(button._pressTimer);
  button._pressTimer = setTimeout(() => button.classList.remove('is-pressed'), 160);
};

const renderStats = (data) => {
  const sortedDates = Object.keys(data).sort();
  const todayKey = currentDateKey();
  const today = getTodaySummary(data, todayKey);

  const correctSeries = sortedDates.map((dateKey) => safeNumber(data[dateKey]?.correct));
  const wrongSeries = sortedDates.map((dateKey) => safeNumber(data[dateKey]?.wrong));

  const correctSummary = getTrendSummary(correctSeries);
  const wrongSummary = getTrendSummary(wrongSeries);

  const totalCorrect = correctSeries.reduce((sum, value) => sum + value, 0);
  const totalWrong = wrongSeries.reduce((sum, value) => sum + value, 0);

  const correctMeter = document.getElementById('totalCorrect');
  const wrongMeter = document.getElementById('totalWrong');
  const correctTrend = document.getElementById('correctTrend');
  const wrongTrend = document.getElementById('wrongTrend');
  const todaySummary = document.getElementById('todaySummary');
  const todayComparison = document.getElementById('todayComparison');

  correctMeter.textContent = String(totalCorrect);
  wrongMeter.textContent = String(totalWrong);

  correctTrend.textContent = `${correctSummary.percentage}% all-time trend`;
  wrongTrend.textContent = `${wrongSummary.percentage}% all-time trend`;

  correctTrend.classList.toggle('positive', correctSummary.percentage >= 0);
  correctTrend.classList.toggle('negative', correctSummary.percentage < 0);
  wrongTrend.classList.toggle('positive', wrongSummary.percentage >= 0);
  wrongTrend.classList.toggle('negative', wrongSummary.percentage < 0);

  todaySummary.textContent = `${today.correct} / ${today.wrong}`;

  if (sortedDates.length > 1) {
    const previousDate = sortedDates[sortedDates.length - 2];
    const previousToday = getTodaySummary(data, previousDate);
    const correctDelta = calculatePercentageChange(previousToday.correct, today.correct);
    const wrongDelta = calculatePercentageChange(previousToday.wrong, today.wrong);

    todayComparison.textContent = `Correct ${correctDelta}% • Wrong ${wrongDelta}% vs. yesterday`;
    todayComparison.classList.toggle('positive', correctDelta >= 0 || wrongDelta >= 0);
    todayComparison.classList.toggle('negative', correctDelta < 0 || wrongDelta < 0);
  } else {
    todayComparison.textContent = 'No trend yet';
    todayComparison.classList.remove('positive', 'negative');
  }

  const correctCount = document.getElementById('correctCount');
  const wrongCount = document.getElementById('wrongCount');
  correctCount.textContent = String(today.correct);
  wrongCount.textContent = String(today.wrong);
};

const renderChart = (data) => {
  const labels = Object.keys(data).sort().map(formatDateLabel);
  const correctValues = labels.map((_, index) => safeNumber(Object.keys(data).sort()[index] ? data[Object.keys(data).sort()[index]].correct : 0));
  const wrongValues = labels.map((_, index) => safeNumber(Object.keys(data).sort()[index] ? data[Object.keys(data).sort()[index]].wrong : 0));

  const canvas = document.getElementById('trendChart');
  const existingChart = Chart.getChart(canvas);
  if (existingChart) {
    existingChart.destroy();
  }

  new Chart(canvas, {
    type: 'line',
    data: {
      labels,
      datasets: [
        {
          label: 'Correct',
          data: correctValues,
          borderColor: '#22c55e',
          backgroundColor: 'rgba(34, 197, 94, 0.18)',
          pointBackgroundColor: '#22c55e',
          pointBorderColor: '#ffffff',
          pointRadius: 4,
          pointHoverRadius: 5,
          borderWidth: 3,
          tension: 0.3,
          fill: false
        },
        {
          label: 'Wrong',
          data: wrongValues,
          borderColor: '#ef4444',
          backgroundColor: 'rgba(239, 68, 68, 0.15)',
          pointBackgroundColor: '#ef4444',
          pointBorderColor: '#ffffff',
          pointRadius: 4,
          pointHoverRadius: 5,
          borderWidth: 3,
          tension: 0.3,
          fill: false
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      interaction: {
        mode: 'nearest',
        intersect: false
      },
      plugins: {
        legend: {
          labels: {
            color: '#e2e8f0',
            usePointStyle: true,
            pointStyle: 'circle',
            boxWidth: 10
          }
        },
        tooltip: {
          backgroundColor: 'rgba(15, 23, 42, 0.92)',
          titleColor: '#f8fafc',
          bodyColor: '#f8fafc',
          borderColor: 'rgba(148, 163, 184, 0.2)',
          borderWidth: 1
        }
      },
      scales: {
        x: {
          ticks: { color: '#cbd5e1' },
          grid: { color: 'rgba(148, 163, 184, 0.12)' }
        },
        y: {
          beginAtZero: true,
          ticks: { color: '#cbd5e1' },
          grid: { color: 'rgba(148, 163, 184, 0.12)' }
        }
      }
    }
  });
};

const renderDashboard = () => {
  const data = loadData();
  ensureTodayEntry(data);
  saveData(data);
  renderStats(data);
  renderChart(data);
};

const incrementButton = (type) => {
  const data = loadData();
  const trackedDay = ensureTodayEntry(data);
  trackedDay[type] = safeNumber(trackedDay[type]) + 1;
  saveData(data);
  renderDashboard();
};

const resetData = () => {
  const confirmed = window.confirm('Clear all saved tracker data?');
  if (!confirmed) return;
  localStorage.removeItem(STORAGE_KEY);
  renderDashboard();
};

document.addEventListener('DOMContentLoaded', () => {
  const correctButton = document.getElementById('correctBtn');
  const wrongButton = document.getElementById('wrongBtn');
  const clearDataBtn = document.getElementById('clearDataBtn');

  correctButton.addEventListener('pointerdown', () => setButtonPressedState(correctButton));
  wrongButton.addEventListener('pointerdown', () => setButtonPressedState(wrongButton));

  correctButton.addEventListener('click', () => incrementButton('correct'));
  wrongButton.addEventListener('click', () => incrementButton('wrong'));
  clearDataBtn.addEventListener('click', resetData);

  renderDashboard();
});

window.addEventListener('storage', renderDashboard);
