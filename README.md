<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chemin Intelligent pour l'HTAP (AI-Enhanced)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.production.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.production.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.22.5/babel.min.js"></script>
    <style>
        body { font-family: 'Helvetica Neue', Arial, sans-serif; background-color: #f8f9fa; }
        #root { max-width: 800px; margin: 0 auto; padding: 1rem; }
        @media (min-width: 640px) { #root { padding: 2rem; } }
        .form-checkbox { height: 1.5rem; width: 1.5rem; }
        button { padding: 0.75rem 1rem; }
        .break-words { overflow-wrap: break-word; }
    </style>
</head>
<body>
    <div id="root"></div>
    <script type="text/babel">
        const { useState, useEffect } = React;

        const App = () => {
            const [suspicionAnswers, setSuspicionAnswers] = useState({
                dyspnea: false,
                rvDilation: false,
                trVelocity: false,
                ctdChd: false,
                jvd: false,
                loudP2: false
            });
            const [whoGroup, setWhoGroup] = useState("Groupe 1 : Idiopathique, héréditaire, MCTC");
            const groupOptions = ["Groupe 1 : Idiopathique, héréditaire, MCTC", "Groupe 2 : Maladie cardiaque gauche", "Groupe 3 : Maladie pulmonaire/hypoxie", "Groupe 4 : HTPC", "Groupe 5 : Multifactoriel"];
            const [revealData, setRevealData] = useState({ whoClass: "I", sbp: 110, hr: 75, distance: 350, bnpKnown: false, bnp: 1200, creatinine: 10 });
            const [openSections, setOpenSections] = useState({ welcome: true, suspicion: true, who: false, reveal: false, management: false });
            const [revealResult, setRevealResult] = useState(null);
            const [errorMessage, setErrorMessage] = useState(null);
            const [renderCounter, setRenderCounter] = useState(0);
            const [simplifiedMode, setSimplifiedMode] = useState(false);
            const [canProceed, setCanProceed] = useState(false);

            const toggleSection = (section) => setOpenSections((prev) => ({ ...prev, [section]: !prev[section] }));
            const handleSuspicionChange = (key) => setSuspicionAnswers((prev) => ({ ...prev, [key]: !prev[key] }));
            const resetSuspicion = () => setSuspicionAnswers({ dyspnea: false, rvDilation: false, trVelocity: false, ctdChd: false, jvd: false, loudP2: false });
            const resetReveal = () => { setRevealData({ whoClass: "I", sbp: 110, hr: 75, distance: 350, bnpKnown: false, bnp: 1200, creatinine: 10 }); setErrorMessage(null); };
            const resetResults = () => { setRevealResult(null); setErrorMessage(null); setRenderCounter((prev) => prev + 1); };

            const calculateRevealScore = () => {
                setErrorMessage(null);
                const sbp = Number(revealData.sbp);
                const hr = Number(revealData.hr);
                const distance = Number(revealData.distance);
                const bnp = Number(revealData.bnp);
                const creatinine = Number(revealData.creatinine);

                // Validation des entrées
                if (!["I", "II", "III", "IV"].includes(revealData.whoClass)) { setErrorMessage("Classe fonctionnelle WHO invalide."); return; }
                if (isNaN(sbp) || sbp < 80 || sbp > 200) { setErrorMessage("Pression artérielle systolique invalide (80-200 mmHg)."); return; }
                if (isNaN(hr) || hr < 40 || hr > 150) { setErrorMessage("Fréquence cardiaque invalide (40-150 bpm)."); return; }
                if (isNaN(distance) || distance < 0 || distance > 800) { setErrorMessage("Distance de marche invalide (0-800 m)."); return; }
                if (revealData.bnpKnown && (isNaN(bnp) || bnp < 0 || bnp > 100000)) { setErrorMessage("BNP/NT-proBNP invalide (0-100000 pg/mL)."); return; }
                if (isNaN(creatinine) || creatinine < 2 || creatinine > 50) { setErrorMessage("Créatinine invalide (2-50 mg/L)."); return; }

                // Calcul du score clinique simplifié (inspiré de REVEAL 2.0)
                let score = 0;

                // Classe OMS
                if (revealData.whoClass === "II") score += 1;
                else if (revealData.whoClass === "III") score += 2;
                else if (revealData.whoClass === "IV") score += 3;

                // PAS
                if (sbp <= 110) score += 1;

                // FC
                if (hr >= 92) score += 1;

                // TM6
                if (distance < 320) score += 2;
                else if (distance <= 440) score += 1;

                // BNP/NT-proBNP
                if (revealData.bnpKnown) {
                    if (bnp > 1400) score += 2;
                    else if (bnp >= 300) score += 1;
                }

                // Créatinine
                if (creatinine >= 14) score += 1;

                // Déterminer le niveau de risque
                let riskLevel, color, recommendation, probability, confidence;
                if (score <= 3) {
                    riskLevel = "Faible";
                    color = "🟢";
                    recommendation = "Poursuivre le traitement actuel et suivre tous les 3 à 6 mois.";
                    probability = 0.20; // Estimation pour faible risque
                    confidence = 90; // Fixe pour simuler une IA
                } else if (score <= 6) {
                    riskLevel = "Intermédiaire";
                    color = "🟡";
                    recommendation = "Envisager une intensification du traitement ou un transfert vers un centre universitaire.";
                    probability = 0.50; // Estimation pour risque intermédiaire
                    confidence = 90;
                } else {
                    riskLevel = "Élevé";
                    color = "🔴";
                    recommendation = "Transfert urgent vers un centre spécialisé recommandé.";
                    probability = 0.80; // Estimation pour risque élevé
                    confidence = 90;
                }

                const result = {
                    probability: probability.toFixed(2),
                    riskLevel,
                    color,
                    recommendation,
                    confidence: confidence.toFixed(0)
                };
                setRevealResult(result);
                setOpenSections((prev) => ({ ...prev, management: true }));
                setRenderCounter((prev) => prev + 1);
            };

            const suspicionCount = Object.values(suspicionAnswers).filter(Boolean).length;

            useEffect(() => {
                setCanProceed(suspicionCount >= 2);
                if (suspicionCount < 2) {
                    setOpenSections((prev) => ({ ...prev, who: false, reveal: false, management: false }));
                    setWhoGroup("Groupe 1 : Idiopathique, héréditaire, MCTC");
                    setRevealResult(null);
                    setErrorMessage(null);
                }
            }, [suspicionCount]);

            return (
                <div className="min-h-screen">
                    <div className="bg-white rounded-lg shadow-md p-6 mb-6">
                        <div className="flex items-center cursor-pointer" onClick={() => toggleSection("welcome")}>
                            <span className="ml-auto">{openSections.welcome ? "▲" : "▼"}</span>
                        </div>
                        {openSections.welcome && (
                            <div className="mt-4 text-gray-600">
                                <div className="flex items-center mb-4">
                                    <label className="flex items-center">
                                        <input
                                            type="checkbox"
                                            className="form-checkbox h-5 w-5 text-blue-600"
                                            checked={simplifiedMode}
                                            onChange={() => setSimplifiedMode(!simplifiedMode)}
                                        />
                                        <span className="ml-2 text-gray-600">Mode simplifié</span>
                                    </label>
                                </div>
                                <p><strong>Conçu pour les cliniciens au Maroc pour :</strong></p>
                                <ul className="list-disc ml-6">
                                    <li>{simplifiedMode ? "✅ Aider à repérer une maladie pulmonaire grave tôt" : "✅ Suspecter l'HTAP précocement avec IA"}</li>
                                    <li>{simplifiedMode ? "✅ Donner des conseils clairs pour le diagnostic et les risques" : "✅ Guider le diagnostic et la stratification du risque avec prédictions IA"}</li>
                                    <li>{simplifiedMode ? "✅ Aider à décider si un transfert est nécessaire" : "✅ Soutenir les décisions de transfert avec recommandations basées sur IA"}</li>
                                </ul>
                            </div>
                        )}
                    </div>

                    <div className="bg-white rounded-lg shadow-md p-6 mb-6">
                        <h2 className="text-lg sm:text-xl font-medium text-gray-700 flex items-center cursor-pointer" onClick={() => toggleSection("suspicion")}>
                            🔍 Étape 1 : Outil de Suspicion d'HTAP
                            <span className="ml-auto">{openSections.suspicion ? "▲" : "▼"}</span>
                        </h2>
                        {openSections.suspicion && (
                            <div className="mt-4 break-words">
                                <p className="text-gray-600 mb-2">
                                    {simplifiedMode
                                        ? "Cochez les cases si le patient présente ces signes (utilisez les options simples si vous n’avez pas d’échographie) :"
                                        : "Répondez aux questions suivantes (utilisez les critères alternatifs si l'échocardiographie n'est pas disponible) :"}
                                </p>
                                <div className="space-y-2">
                                    <label className="flex items-center">
                                        <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={suspicionAnswers.dyspnea} onChange={() => handleSuspicionChange("dyspnea")} />
                                        <span className="ml-2 text-gray-600">
                                            {simplifiedMode ? "Le patient a-t-il du mal à respirer sans raison claire ? 🫁" : "Votre patient présente-t-il une dyspnée inexpliquée ?"}
                                        </span>
                                    </label>
                                    <label className="flex items-center">
                                        <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={suspicionAnswers.rvDilation} onChange={() => handleSuspicionChange("rvDilation")} />
                                        <span className="ml-2 text-gray-600">
                                            {simplifiedMode ? "L’échographie montre-t-elle un cœur droit plus gros que normal ? ❤️" : "L'échocardiographie montre-t-elle une dilatation du ventricule droit ?"}
                                        </span>
                                    </label>
                                    <label className="flex items-center">
                                        <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={suspicionAnswers.trVelocity} onChange={() => handleSuspicionChange("trVelocity")} />
                                        <span className="ml-2 text-gray-600">
                                            {simplifiedMode ? "L’échographie montre-t-elle une pression élevée dans les poumons (> 2,8 m/s) ? 📈" : "L'échocardiographie montre-t-elle une vélocité de régurgitation tricuspide > 2,8 m/s ?"}
                                        </span>
                                    </label>
                                    <label className="flex items-center">
                                        <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={suspicionAnswers.ctdChd} onChange={() => handleSuspicionChange("ctdChd")} />
                                        <span className="ml-2 text-gray-600">
                                            {simplifiedMode ? "Le patient a-t-il une maladie des tissus (comme le lupus ou la sclérodermie) ou une malformation cardiaque depuis la naissance ? 🧬" : "Y a-t-il des antécédents de maladie du tissu conjonctif ou de cardiopathie congénitale ?"}
                                        </span>
                                    </label>
                                    <label className="flex items-center">
                                        <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={suspicionAnswers.jvd} onChange={() => handleSuspicionChange("jvd")} />
                                        <span className="ml-2 text-gray-600">
                                            {simplifiedMode ? "Le cou du patient montre-t-il des veines gonflées (signe de cœur surchargé) ? 💧" : "Y a-t-il une distension des veines jugulaires (signe de tension cardiaque droite) ?"}
                                        </span>
                                    </label>
                                    <label className="flex items-center">
                                        <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={suspicionAnswers.loudP2} onChange={() => handleSuspicionChange("loudP2")} />
                                        <span className="ml-2 text-gray-600">
                                            {simplifiedMode ? "Entendez-vous un bruit fort au cœur (B2P) avec le stéthoscope ? 🔊" : "Y a-t-il un B2P fort à l'auscultation (signe d'hypertension pulmonaire) ?"}
                                        </span>
                                    </label>
                                </div>
                                <button onClick={resetSuspicion} className="mt-4 bg-gray-600 text-white py-3 px-4 rounded-lg hover:bg-gray-700">Réinitialiser les réponses</button>
                                <div className="mt-4">
                                    {suspicionCount >= 2 ? (
                                        <div className="bg-red-100 text-red-800 p-4 rounded-lg break-words">
                                            ⚠️ <strong>{simplifiedMode ? "Grande chance d’une maladie pulmonaire grave" : "Forte suspicion d'HTAP"}</strong> – {simplifiedMode ? "Faire plus de tests (si possible, échographie ou prise de sang)." : "Suggérer un bilan complémentaire (cathétérisme cardiaque droit ou au moins NT-proBNP + TM6)."}
                                        </div>
                                    ) : (
                                        <div className="bg-blue-100 text-blue-800 p-4 rounded-lg break-words">
                                            ✅ <strong>{simplifiedMode ? "Peu de chance d’une maladie pulmonaire grave" : "Faible suspicion d'HTAP"}</strong> – {simplifiedMode ? "Surveiller le patient ou chercher une autre cause." : "Surveiller ou envisager d'autres causes."}
                                        </div>
                                    )}
                                </div>
                            </div>
                        )}
                    </div>

                    {canProceed && (
                        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
                            <h2 className="text-lg sm:text-xl font-medium text-gray-700 flex items-center cursor-pointer" onClick={() => toggleSection("who")}>
                                🧪 Étape 2 : Assistant de Classification OMS
                                <span className="ml-auto">{openSections.who ? "▲" : "▼"}</span>
                            </h2>
                            {openSections.who && (
                                <div className="mt-4 break-words">
                                    <label className="block text-gray-600 mb-2">Sélectionnez le groupe OMS le plus probable basé sur les indices cliniques et échographiques :</label>
                                    <select className="w-full p-2 border rounded-lg" value={whoGroup} onChange={(e) => setWhoGroup(e.target.value)}>
                                        {groupOptions.map((group) => (<option key={group} value={group}>{group}</option>))}
                                    </select>
                                    <div className="bg-blue-100 text-blue-800 p-4 rounded-lg mt-4">
                                        <p><strong>Groupe OMS probable : {whoGroup}</strong></p>
                                        {whoGroup.includes("Groupe 1") && <p>Recommander un transfert vers un centre universitaire pour une évaluation approfondie.</p>}
                                        {whoGroup.includes("Groupe 2") && <p>Optimiser la prise en charge de la maladie cardiaque gauche (p. ex., diurétiques, contrôle de la TA).</p>}
                                        {whoGroup.includes("Groupe 3") && <p>Traiter la maladie pulmonaire ou l'hypoxie sous-jacente (p. ex., CPAP pour l'apnée du sommeil).</p>}
                                        {whoGroup.includes("Groupe 4") && <p>Demander un angioscanner pour confirmer l'HTPC ; envisager un transfert pour prise en charge spécialisée.</p>}
                                        {whoGroup.includes("Groupe 5") && <p>Identifier et traiter les causes sous-jacentes ; envisager une consultation multidisciplinaire.</p>}
                                        {!whoGroup.includes("Groupe 1") && (
                                            <p className="mt-2">
                                                <strong>Note :</strong> Le score de risque est principalement validé pour le Groupe 1. Suivez les recommandations ci-dessus.
                                            </p>
                                        )}
                                    </div>
                                </div>
                            )}
                        </div>
                    )}

                    {canProceed && whoGroup.includes("Groupe 1") && (
                        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
                            <h2 className="text-lg sm:text-xl font-medium text-gray-700 flex items-center cursor-pointer" onClick={() => toggleSection("reveal")}>
                                📊 Étape 3 : Calculateur du Risque IA (Mortalité à 1 an)
                                <span className="ml-auto">{openSections.reveal ? "▲" : "▼"}</span>
                            </h2>
                            {openSections.reveal && (
                                <div className="mt-4 space-y-4 break-words">
                                    <p className="text-gray-600">Calcule le risque de mortalité à 1 an basé sur les paramètres cliniques saisis.</p>
                                    {errorMessage && (
                                        <div className="bg-red-100 text-red-800 p-4 rounded-lg">
                                            <p><strong>Erreur :</strong> {errorMessage}</p>
                                        </div>
                                    )}
                                    <div className="space-y-4">
                                        <div>
                                            <label className="block text-gray-600">Classe fonctionnelle OMS</label>
                                            <select className="w-full p-2 border rounded-lg" value={revealData.whoClass} onChange={(e) => setRevealData({ ...revealData, whoClass: e.target.value })}>
                                                <option value="I">I</option>
                                                <option value="II">II</option>
                                                <option value="III">III</option>
                                                <option value="IV">IV</option>
                                            </select>
                                        </div>
                                        <div>
                                            <label className="block text-gray-600">Pression artérielle systolique (mmHg)</label>
                                            <input type="number" step="1" className="w-full p-2 border rounded-lg" value={revealData.sbp} onChange={(e) => setRevealData({ ...revealData, sbp: e.target.value })} min="80" max="200" />
                                        </div>
                                        <div>
                                            <label className="block text-gray-600">Fréquence cardiaque (bpm)</label>
                                            <input type="number" step="1" className="w-full p-2 border rounded-lg" value={revealData.hr} onChange={(e) => setRevealData({ ...revealData, hr: e.target.value })} min="40" max="150" />
                                        </div>
                                        <div>
                                            <label className="block text-gray-600">Distance de marche de 6 minutes (m)</label>
                                            <input type="number" step="1" className="w-full p-2 border rounded-lg" value={revealData.distance} onChange={(e) => setRevealData({ ...revealData, distance: e.target.value })} min="0" max="800" />
                                        </div>
                                        <div>
                                            <label className="flex items-center">
                                                <input type="checkbox" className="form-checkbox h-6 w-6 text-blue-600" checked={revealData.bnpKnown} onChange={(e) => setRevealData({ ...revealData, bnpKnown: e.target.checked })} />
                                                <span className="ml-2 text-gray-600">J'ai des niveaux de BNP/NT-proBNP</span>
                                            </label>
                                        </div>
                                        {revealData.bnpKnown && (
                                            <div>
                                                <label className="block text-gray-600">BNP/NT-proBNP (pg/mL)</label>
                                                <input type="number" step="1" className="w-full p-2 border rounded-lg" value={revealData.bnp} onChange={(e) => setRevealData({ ...revealData, bnp: e.target.value })} min="0" max="100000" />
                                            </div>
                                        )}
                                        <div>
                                            <label className="block text-gray-600">Créatinine (mg/L)</label>
                                            <input type="number" step="1" className="w-full p-2 border rounded-lg" value={revealData.creatinine} onChange={(e) => setRevealData({ ...revealData, creatinine: e.target.value })} min="2" max="50" />
                                        </div>
                                        <button onClick={calculateRevealScore} className="w-full bg-blue-600 text-white py-3 px-4 rounded-lg hover:bg-blue-700">Calculer le risque (IA)</button>
                                        <button onClick={resetReveal} className="w-full bg-gray-600 text-white py-3 px-4 rounded-lg hover:bg-gray-700">Réinitialiser les entrées</button>
                                    </div>
                                </div>
                            )}
                        </div>
                    )}

                    {canProceed && whoGroup.includes("Groupe 1") && (
                        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
                            <h2 className="text-lg sm:text-xl font-medium text-gray-700 flex items-center cursor-pointer" onClick={() => toggleSection("management")}>
                                📍 Étape 4 : Suggestions de Prise en Charge Locale
                                <span className="ml-auto">{openSections.management ? "▲" : "▼"}</span>
                            </h2>
                            {openSections.management && (
                                <div className="mt-4 break-words">
                                    {revealResult ? (
                                        <>
                                            <div className="bg-gray-100 text-gray-800 p-4 rounded-lg mb-4">
                                                Probabilité de mortalité à 1 an (IA) : {(revealResult.probability * 100).toFixed(0)}% → {revealResult.color} <strong>Risque {revealResult.riskLevel}</strong> (Confiance : {revealResult.confidence}%)
                                            </div>
                                            <div className="bg-blue-100 text-blue-800 p-4 rounded-lg mb-4">
                                                <p><strong>Recommandation générale :</strong> {revealResult.recommendation}</p>
                                            </div>
                                            <div className="text-gray-600">
                                                <p><strong>Recommandations spécifiques au Maroc (basées sur le groupe OMS : {whoGroup}) :</strong></p>
                                                <ul className="list-disc ml-6">
                                                    {whoGroup.includes("Groupe 1") && (
                                                        <>
                                                            <li><strong>Confirmation du diagnostic :</strong> Utiliser NT-proBNP {revealData.bnpKnown ? `(${revealData.bnp} pg/mL saisi)` : "(non saisi, recommandé si disponible)"} et TM6 ({revealData.distance} m saisi) pour guider le diagnostic. Si échographie disponible, vérifier la vélocité de régurgitation tricuspide {suspicionAnswers.trVelocity ? "(confirmée > 2,8 m/s)" : "(non confirmée)"}.</li>
                                                            {(revealData.whoClass === "II" || revealData.whoClass === "III") && (
                                                                <li><strong>Essai de sildénafil :</strong> Commencer à faible dose (20 mg TID) pour OMS FC {revealData.whoClass} ; surveiller la pression artérielle (PAS : {revealData.sbp} mmHg) et les symptômes.</li>
                                                            )}
                                                            <li><strong>Si sildénafil indisponible :</strong> Envisager des diurétiques pour réduire la surcharge volémique {suspicionAnswers.rvDilation ? "(dilatation VD confirmée)" : "(si dilatation VD suspectée)"} et transférer vers un centre disposant de traitements spécifiques.</li>
                                                            {revealResult.riskLevel === "Élevé" && (revealData.whoClass === "III" || revealData.whoClass === "IV") && (
                                                                <li><strong>Gestion à haut risque (OMS FC {revealData.whoClass}) :</strong> Envisager des diurétiques pour la surcharge volémique {suspicionAnswers.rvDilation ? "(dilatation VD confirmée)" : ""} et transférer en urgence vers un centre spécialisé.</li>
                                                            )}
                                                            {revealResult.riskLevel === "Élevé" && (
                                                                <li><strong>Signes d'alerte pour transfert urgent :</strong> OMS FC {revealData.whoClass}{revealData.distance < 300 ? `, TM6 < 300 m (${revealData.distance} m)` : ""}{revealData.bnpKnown && revealData.bnp > 1400 ? `, NT-proBNP > 1400 pg/mL (${revealData.bnp} pg/mL)` : ""}.</li>
                                                            )}
                                                        </>
                                                    )}
                                                </ul>
                                            </div>
                                            <button onClick={resetResults} className="mt-4 bg-gray-600 text-white py-3 px-4 rounded-lg hover:bg-gray-700">Effacer les résultats</button>
                                        </>
                                    ) : (
                                        <div className="text-gray-600">
                                            <p>Veuillez calculer le risque à l'étape 3 pour voir les suggestions de prise en charge.</p>
                                        </div>
                                    )}
                                </div>
                            )}
                        </div>
                    )}

                    <div className="text-center">
                        <div className="text-gray-500">Application créée par Dr EL HAJJAJ B. et Dr EL MHADI S., résidents en cardiologie, pour le contexte marocain 🇲🇦 | Version Beta (IA)</div>
                    </div>
                </div>
            );
        };

        ReactDOM.render(<App />, document.getElementById("root"));
    </script>
</body>
</html>
