function D = DO_MPPT(V, I)
%#codegen
% DO_MPPT - Maximum Power Point Tracking menggunakan Dandelion Optimizer
% Input: V (tegangan), I (arus)
% Output: D (duty cycle optimal, skalar double)
%
% Fungsi evaluateMPPT mengukur fitness duty cycle. Di sini digunakan fungsi dummy
% yang memaksimalkan pada D = 0.5.

persistent Population Fitness t Elite Pop_size Max_iterations T ...
    alpha beta delta a b c LB UB dim

% Deklarasikan ukuran maksimum untuk variabel persistennya
coder.varsize('Population', [100, 1], [true, false]);
coder.varsize('Fitness',    [100, 1], [true, false]);
coder.varsize('Elite',      [1, 1], [false, false]);

% Pastikan output D berupa skalar
D = 0;

if isempty(Population)
    %% Inisialisasi Parameter
    Pop_size = 100;       % Ukuran populasi
    Max_iterations = 100; % Maksimum iterasi
    T = Max_iterations;  % Untuk perhitungan parameter adaptif
    dim = 1;             % Dimensi masalah (1D: duty cycle)
    LB = 0;              % Batas bawah duty cycle
    UB = 1;              % Batas atas duty cycle
    
    % Inisialisasi populasi awal secara acak dalam [LB, UB]
    Population = LB + (UB - LB).*rand(Pop_size, dim);
    
    % Evaluasi fitness awal
    Fitness = zeros(Pop_size, 1);
    for i = 1:Pop_size
        D_candidate = Population(i);
        Fitness(i) = evaluateMPPT(D_candidate);
    end
    
    % Tentukan solusi elite awal
    [~, idx] = max(Fitness);
    Elite = Population(idx, :);
    t = 1;
    
    % Parameter algoritma sesuai paper
    beta = 1.5;    % Parameter untuk levy flight (eq. 16)
    delta = 2;     % Parameter pada landing stage (eq. 15)
    a = 0.3;       % Koefisien untuk parameter k (eq. 11)
    b = 0.4;
    c = 0.3;
end

%% Perhitungan Parameter Adaptif
alpha = rand() * ((1/T^2)*t^2 - (2/T)*t + 1);  % (eq. 8)
k = 1 - rand()*(a*(t/T)^2 + b*(t/T) + c);        % (eq. 11)

%% Rising Stage (Eksplorasi)
RisingPop = zeros(Pop_size, dim);
for i = 1:Pop_size
    if randn() < 1.5  % Kondisi cuaca cerah: eksplorasi global
        Y = lognrnd(0, 1);  % Distribusi log-normal
        X_rand = Population(randi(Pop_size), :);
        theta = (2*rand()-1)*pi;
        v_x = cos(theta);
        v_y = sin(theta);
        % Update posisi sesuai persamaan (5)
        RisingPop(i, :) = Population(i, :) + alpha * v_x * v_y * log(Y) * (X_rand - Population(i, :));
    else
        % Kondisi hujan: eksploitasi lokal (eq. 10)
        RisingPop(i, :) = Population(i, :) * k;
    end
end
RisingPop = max(min(RisingPop, UB), LB);

%% Decline Stage (Eksploitasi Global)
MeanPosition = mean(Population);
DeclinePop = zeros(Pop_size, dim);
for i = 1:Pop_size
    % Update posisi sesuai persamaan (13)
    DeclinePop(i, :) = Population(i, :) - beta * alpha * (MeanPosition - beta*alpha*Population(i, :));
end
DeclinePop = max(min(DeclinePop, UB), LB);

%% Landing Stage (Konvergensi)
LevyStep = levyFlight(Pop_size, dim, beta);
LandingPop = zeros(Pop_size, dim);
for i = 1:Pop_size
    % Update posisi sesuai persamaan (15)
    LandingPop(i, :) = Elite + delta * LevyStep(i, :) .* (Elite - Population(i, :) * (2*t/T));
end
LandingPop = max(min(LandingPop, UB), LB);

%% Gabungkan dan Evaluasi Populasi Baru
NewPopulation = [RisingPop; DeclinePop; LandingPop]; % Ukuran: 45x1
NewFitness = zeros(size(NewPopulation,1), 1);
for i = 1:size(NewPopulation,1)
    D_candidate = NewPopulation(i, :);
    NewFitness(i) = evaluateMPPT(D_candidate);
end

% Gabungkan populasi lama dan baru, lalu seleksi elitism
CombinedPop = [Population; NewPopulation];         % Ukuran: 60x1
CombinedFitness = [Fitness; NewFitness];             % Ukuran: 60x1
[~, sortedIdx] = sort(CombinedFitness, 'descend');
Population = CombinedPop(sortedIdx(1:Pop_size), :);   % Kembali menjadi 15x1
Fitness = CombinedFitness(sortedIdx(1:Pop_size));

% Update solusi elite
[~, idx] = max(Fitness);
Elite = Population(idx, :);

% Output duty cycle optimal sebagai skalar
D = Elite(1);
D = reshape(D, [1, 1]);

% Update iterasi
t = t + 1;
if t > Max_iterations
    t = 1;
end

end

%% Fungsi Levy Flight (sesuai eq. 16)
function L = levyFlight(n, d, beta)
    sigma = (gamma(1+beta)*sin(pi*beta/2) / (gamma((1+beta)/2)*beta*2^((beta-1)/2)))^(1/beta);
    u = randn(n, d)*sigma;
    v = randn(n, d);
    L = 0.01 * u ./ (abs(v).^(1/beta));
end

%% Fungsi Evaluasi MPPT
function power = evaluateMPPT(D)
    % Fungsi evaluasi duty cycle (contoh dummy):
    % Maksimum pada D = 0.5, misalnya P = -(D-0.5)^2 + 1
    power = - (D - 0.5)^2 + 1;
end
