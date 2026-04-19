%% Load data
data = readmatrix('/home/hamed/Downloads/Prof. Juhas/4_VOLT.CSV');

t = data(:,1);      % Time (s)
y = data(:,2);      % Voltage (V)

% Create timeseries for Simulink
signal = timeseries(y, t);

figure;
plot(t, y);
xlabel('Time (s)');
ylabel('Voltage (V)');
title('Raw Signal');

%% Basic parameters
Ts = mean(diff(t));   % Sampling period
fs = 1/Ts;            % Sampling frequency
N  = length(y);       % Number of samples

%% Preprocessing
y = y - mean(y);            % Remove DC offset
y = y / max(abs(y));        % Normalize amplitude

%% Low-pass Butterworth filter
fc  = 320;                 % Cutoff frequency (Hz)
Wn  = fc / (fs/2);         % Normalized frequency
ord = 2;                   % Filter order

[b,a] = butter(ord, Wn, 'low');
y_bp  = filtfilt(b, a, y); % Zero-phase filtering
[H,Wn] = freqz(b,a,1024,fs);
%[num,den] = butter(ord,Wn,'s');
freqz(a,b)


% Gain correction
%gain = max(abs(y)) / max(abs(y_bp));
%y_bp = y_bp * gain;

%% Peak detection
[pks, locs] = findpeaks(y_bp,'MinPeakHeight', 0.06,'MinPeakDistance', 56 );   
counter = length(pks);
fprintf('Detected peaks: %d\n', counter);

%% Frequency estimation
sample_distances    = diff(locs);
avg_sample_distance = mean(sample_distances);

T = avg_sample_distance / fs;
fprintf('Time period: %f seconds\n', T);

f = (1 / T) / 3;
fprintf('Frequency: %f Hz\n', f);

omega = 2 * pi * f;
fprintf('W: %f rad/sec\n', omega);
%% Pulse generation (Schmitt trigger)
v_high = 0.06;
v_low  = 0.01;

pulse = zeros(size(y_bp));
state = 0;

for i = 1:length(y_bp)
    if y_bp(i) > v_high
        state = 1;
    elseif y_bp(i) < v_low
        state = 0;
    end
    pulse(i) = state;
end

figure;
subplot(2,1,1);
plot(t,y_bp,'b');
title('Filtered Signal');
xlabel('Time');
ylabel('Amplitude');
grid on;

subplot(2,1,2);
plot(t,pulse,'r','LineWidth',2);
title('Generated pulse');
xlabel('Time');
ylabel('status');
ylim([-0.2 1.2]);
grid on;

%% Final comparison plot
figure;
plot(t, y, 'Color', [0.8 0.8 0]);
hold on;
plot(t, y_bp, 'r', 'LineWidth', 1.4);
legend('Raw', 'Butterworth filtered');
xlabel('Time (s)');
ylabel('Amplitude');
title('Comparison: Raw vs Low-pass Filtered');

figure;plot(Wn,20*log10(abs(H)));grid on;
xlabel('Hz');ylabel('Magnitude(dB)');title('Butterworth frequency response');
