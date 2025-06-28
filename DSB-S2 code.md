function [dvb, encldpcCfg, decldpcCfg] = configureDVBS2Custom_NoBCH(modType, codeRate, numIter, EsNodB)

% --- 根據碼率取得 LDPC 檢查矩陣 ---
H = dvbs2ldpc(codeRate);
ldpcOutLen = double(size(H, 2));
ldpcInLen  = double(ldpcOutLen - size(H, 1));

% --- LDPC Encoder/Decoder 設定 ---
encldpcCfg = ldpcEncoderConfig(H);
decldpcCfg = ldpcDecoderConfig(H);

% --- 調變設定 ---
matchCell = regexp(modType, '\d+', 'match');
modOrder = str2double(matchCell{1});
bitsPerSym = log2(modOrder);
numSyms = ldpcOutLen / bitsPerSym;

% --- SNR 對應的噪聲變異數 ---
noiseVar    = 1 / (2 * bitsPerSym * 10^(EsNodB/10));
noiseVarEst = noiseVar * 2.56; % 可依情況微調

% --- 交錯順序 ---
interleaveOrder = randperm(ldpcOutLen);

% --- 組合參數結構 dvb ---
dvb = struct();
dvb.CodeRate              = codeRate;
dvb.EsNodB                = EsNodB;
dvb.ModulationType        = modType;
dvb.LDPCParityCheckMatrix = H;
dvb.LDPCCodewordLength    = ldpcOutLen;
dvb.NumInfoBitsPerCodeword = ldpcInLen;
dvb.LDPCNumIterations     = numIter;
dvb.InterleaveOrder       = interleaveOrder;
dvb.ModulationOrder       = modOrder;
dvb.BitsPerSymbol         = bitsPerSym;
dvb.NumSymsPerCodeword    = numSyms;
dvb.NoiseVar              = noiseVar;
dvb.NoiseVarEst           = noiseVarEst;
dvb.NumPacketsPerBBFrame  = 28;  % 任意調整
dvb.CodeID                = 's2';  % 保留但不使用
dvb.PhaseOffset           = 0;
dvb.SymbolMapping         = 'Gray';

end


[dvb, encldpcCfg, decldpcCfg] = configureDVBS2Custom('4PSK', 1/2, 200, -9);

numFrames = 20;
numIterVec = zeros(numFrames,1);
falseVec   = false(dvb.NumPacketsPerBBFrame,1);

for frameCnt=1:numFrames

    % Transmitter, channel, and receiver
    bbFrameTx = logical(randi([0 1],dvb.NumInfoBitsPerCodeword,1));

    ldpcEncOut = ldpcEncode(bbFrameTx,encldpcCfg);
    intrlvrOut = intrlv(ldpcEncOut,dvb.InterleaveOrder);

    if dvb.ModulationOrder == 4 || dvb.ModulationOrder == 8
        modOut = pskmod(intrlvrOut, dvb.ModulationOrder, dvb.PhaseOffset, ...
            dvb.SymbolMapping, InputType="Bit");
    else
        modOut = dvbsapskmod(intrlvrOut,dvb.ModulationOrder,dvb.CodeID, ...
            dvb.CodeRate,'InputType','bit','UnitAveragePower',true);
    end

    chanOut = chan(modOut);

    if dvb.ModulationOrder == 4 || dvb.ModulationOrder == 8
        demodOut = pskdemod(chanOut, dvb.ModulationOrder, dvb.PhaseOffset, ...
            dvb.SymbolMapping, OutputType="approxllr", NoiseVariance=dvb.NoiseVar);
    else
        demodOut = dvbsapskdemod(chanOut,dvb.ModulationOrder,dvb.CodeID, ...
            dvb.CodeRate,'OutputType','approxllr','NoiseVar', ...
            dvb.NoiseVar,'UnitAveragePower',true);
    end

    deintrlvrOut = deintrlv(demodOut,dvb.InterleaveOrder);
    [ldpcDecOut, numIter] = ldpcDecode(deintrlvrOut,decldpcCfg,dvb.LDPCNumIterations);
    bbFrameRx = ldpcDecOut(1:dvb.NumInfoBitsPerCodeword,1);

    % Error statistics
    comparedBits = xor(bbFrameRx, bbFrameTx);
    bitErrors    = sum(comparedBits);
    berLDPC      = bitErrors / dvb.NumInfoBitsPerCodeword;
    berMod = BERMod(demodOut<0,intrlvrOut);
    berLDPC = BERLDPC(logical(ldpcDecOut),bbFrameTx);

    numIterVec(frameCnt) = numIter;
    noiseVar = meanCalc(var(chanOut - modOut));
    constDiag(chanOut);
end

fprintf('Measured SNR : %1.2f dB\n',10*log10(1/noiseVar))
fprintf('Modulator BER: %1.2e\n',berMod(1))
fprintf('LDPC BER     : %1.2e\n',berLDPC(1))
fprintf('PER          : %1.2e\n',per(1))

distFig = figure;
histogram(numIterVec,1:dvb.LDPCNumIterations-1);
xlabel('Number of iterations'); ylabel('# occurrences'); grid on;
title('Distribution of number of LDPC decoder iterations')
]]></w:t></w:r></w:p></w:body></w:document>
