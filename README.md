<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dadaş - YouTube Asistanı</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            text-align: center; 
            padding: 20px;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            min-height: 100vh;
        }
        .container {
            max-width: 800px; margin: 0 auto;
            background: white; border-radius: 20px;
            padding: 30px; box-shadow: 0 10px 40px rgba(0,0,0,0.3);
        }
        h1 { color: #1e3c72; margin-bottom: 10px; }
        .subtitle { color: #666; margin-bottom: 20px; }
        button { 
            padding: 15px 30px; font-size: 18px; cursor: pointer; 
            border: none; border-radius: 10px; color: white; margin: 5px;
            transition: transform 0.2s;
        }
        button:hover { transform: scale(1.05); }
        .btn-baslat { background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); }
        .btn-durdur { background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%); }
        .btn-test { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        select {
            padding: 10px; font-size: 16px; border-radius: 10px; 
            border: 2px solid #1e3c72; width: 100%; max-width: 600px; margin: 10px 0;
        }
        #sonuc { 
            margin: 20px 0; font-size: 20px; color: #333;
            padding: 15px; background: #f0f4f8; border-radius: 10px; min-height: 50px;
        }
        .durum-gosterge {
            display: inline-block; width: 15px; height: 15px;
            border-radius: 50%; margin-right: 10px;
        }
        .aktif { background: #00ff00; box-shadow: 0 0 10px #00ff00; }
        .pasif { background: #ff0000; }
        .sohbet-gecmisi {
            margin-top: 20px; text-align: left;
            max-height: 400px; overflow-y: auto;
            border-top: 2px solid #eee; padding-top: 20px;
        }
        .mesaj {
            margin: 10px 0; padding: 12px; border-radius: 10px;
            max-width: 80%;
        }
        .kullanici { background: #e3f2fd; margin-left: auto; text-align: right; }
        .asistan { background: #f3e5f5; margin-right: auto; text-align: left; }
        .ozellik-listesi {
            margin-top: 20px; padding: 15px;
            background: #fff3cd; border-radius: 10px;
            text-align: left; font-size: 14px;
        }
        .ozellik-listesi h3 { margin-top: 0; color: #856404; }
        .ozellik-listesi ul { margin: 5px 0; padding-left: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🤠 Dadaş - YouTube Asistanı</h1>
        <p class="subtitle">Birtanem'in süper asistanı</p>
        
        <select id="sesSecimi">
            <option value="">Sesler yükleniyor...</option>
        </select>

        <button id="kontrolButon" class="btn-baslat" onclick="toggleDinleme()">
            <span id="butonIkon">🎤</span> <span id="butonYazi">Dinlemeyi Başlat</span>
        </button>
        <button onclick="testSes()" class="btn-test"> Ses Testi</button>
        
        <div id="sonuc">
            <span id="durumIsigi" class="durum-gosterge pasif"></span>
            <span id="durumMetni">Dadaş hazır</span>
        </div>

        <div class="ozellik-listesi">
            <h3>📱 TELEFON & WHATSAPP:</h3>
            <ul>
                <li>📞 <b>"MirecaH ara"</b></li>
                <li> <b>"Oğlum ara"</b> veya <b>"Kızım ara"</b></li>
                <li> <b>"Aybuke ara"</b></li>
                <li>💬 <b>"5551234567'ye WhatsApp'tan yaz"</b></li>
            </ul>
            
            <h3>⏰ HATIRLATICI & NOT:</h3>
            <ul>
                <li> <b>"5 dakika sonra hatırlat"</b></li>
                <li> <b>"Not al: ..."</b></li>
                <li>📋 <b>"Notlarımı oku"</b></li>
            </ul>
            
            <h3>🎬 YOUTUBE:</h3>
            <ul>
                <li>🎥 <b>"Video fikri ver"</b></li>
                <li>📝 <b>"Başlık öner"</b></li>
                <li>📄 <b>"Açıklama yaz"</b></li>
                <li>️ <b>"Etiket ver"</b></li>
            </ul>
        </div>

        <div class="sohbet-gecmisi">
            <div id="gecmis"></div>
        </div>
    </div>

    <script>
        // ⚠️ BURAYA KENDİ API ANAHTARINI YAZ (gsk_... ile başlayan - TIRNAK İŞARETİ İLE!)
        const API_KEY = "gsk_mWOzWQsfUCCaXYCswXA3WGdyb3FYKf8..."; // BURAYA KENDI ANAHTARINI YAZ
        
        let sohbetGecmisi = [];
        let seciliSes = null;
        let recognition = null;
        let dinlemeAktif = false;
        let seslerYuklendiMi = false;
        let asistanKonuşuyor = false; // Feedback loop önleme - KESİN ÇÖZÜM

        // 📱 REHBER
        const rehber = {
            "mirecah": "+905436737439",
            "oglum": "+905419565425",
            "oğlum": "+905419565425",
            "kızım": "+905102217480",
            "kizim": "+905102217480",
            "oğlum müco": "+905528480310",
            "oglum muco": "+905528480310",
            "müco": "+905528480310",
            "muco": "+905528480310",
            "aybuke": "+905467807118"
        };

        // SESLERİ YÜKLE
        function sesleriYukle() {
            if (seslerYuklendiMi) return;
            
            const sesler = speechSynthesis.getVoices();
            if (sesler.length === 0) return;
            
            const secimKutusu = document.getElementById("sesSecimi");
            secimKutusu.innerHTML = "";
            
            let ilkKadinSes = -1;
            let ilkTurkce = -1;
            
            sesler.forEach((ses, index) => {
                const option = document.createElement("option");
                option.value = index;
                option.text = ses.name + " - " + ses.lang;
                
                const isim = ses.name.toLowerCase();
                if (ses.lang.includes('tr') || ses.lang.includes('TR')) {
                    if (ilkKadinSes === -1 && (isim.includes('emel') || isim.includes('natural') || isim.includes('female'))) {
                        ilkKadinSes = index;
                        option.selected = true;
                        seciliSes = ses;
                    }
                    if (ilkTurkce === -1) {
                        ilkTurkce = index;
                    }
                }
                secimKutusu.appendChild(option);
            });
            
            if (ilkKadinSes === -1 && ilkTurkce !== -1) {
                secimKutusu.value = ilkTurkce;
                seciliSes = sesler[ilkTurkce];
            } else if (ilkKadinSes !== -1) {
                secimKutusu.value = ilkKadinSes;
                seciliSes = sesler[ilkKadinSes];
            }
            
            secimKutusu.onchange = function() {
                seciliSes = sesler[this.value];
            };
            
            seslerYuklendiMi = true;
        }

        setTimeout(sesleriYukle, 300);

        // SES TESTİ
        function testSes() {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance("Merhaba Birtanem, Dadaş çalışıyor!");
                utterance.lang = 'tr-TR';
                utterance.rate = 0.9;
                if (seciliSes) utterance.voice = seciliSes;
                window.speechSynthesis.speak(utterance);
            }
        }

        // DİNLEME AÇ/KAPA
        function toggleDinleme() {
            if (dinlemeAktif) durdurDinleme();
            else baslatDinleme();
        }

        function baslatDinleme() {
            var SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            if (!SpeechRecognition) { 
                alert("Chrome veya Edge kullan!"); 
                return; 
            }
            
            recognition = new SpeechRecognition();
            recognition.lang = 'tr-TR';
            recognition.interimResults = false;
            recognition.continuous = true;

            recognition.onstart = function() {
                dinlemeAktif = true;
                document.getElementById("kontrolButon").className = "btn-durdur";
                document.getElementById("butonIkon").innerText = "⏹️";
                document.getElementById("butonYazi").innerText = "Durdur";
                document.getElementById("durumIsigi").className = "durum-gosterge aktif";
                document.getElementById("durumMetni").innerText = "️ Dadaş dinliyor...";
            };

            recognition.onresult = function(event) {
                var ses = event.results[event.results.length - 1][0].transcript;
                
                // SADECE asistan konuşmuyorken işlem yap!
                if (ses.trim() !== "" && !asistanKonuşuyor) {
                    document.getElementById("sonuc").innerHTML = 
                        '<span class="durum-gosterge aktif"></span>🗣️ Dediğin: ' + ses;
                    ekranaYaz("kullanici", ses);
                    sohbetGecmisi.push({role: "user", content: ses});
                    
                    if (!komutKontrol(ses)) {
                        yapayZekayaSor();
                    }
                }
            };

            recognition.onerror = function(event) {
                if (event.error !== 'no-speech') {
                    console.log("Hata:", event.error);
                }
                // İzin hatası olursa durdur
                if (event.error === 'not-allowed') {
                    alert("Mikrofon izni vermen gerekiyor!");
                    durdurDinleme();
                }
            };

            recognition.onend = function() {
                // SADECE asistan konuşmuyorken ve dinleme aktifse tekrar başlat
                if (dinlemeAktif && !asistanKonuşuyor) {
                    setTimeout(() => { 
                        try { 
                            recognition.start(); 
                        } catch(e) {
                            console.log("Mikrofon başlatılamadı:", e);
                        }
                    }, 200);
                }
            };

            try {
                recognition.start();
            } catch(e) {
                alert("Mikrofon başlatılamadı. İzin verdiğinden emin ol!");
            }
        }

        function durdurDinleme() {
            dinlemeAktif = false;
            if (recognition) {
                try { recognition.stop(); } catch(e) {}
            }
            document.getElementById("kontrolButon").className = "btn-baslat";
            document.getElementById("butonIkon").innerText = "";
            document.getElementById("butonYazi").innerText = "Dinlemeyi Başlat";
            document.getElementById("durumIsigi").className = "durum-gosterge pasif";
            document.getElementById("durumMetni").innerText = "Dadaş durdu";
        }

        // KOMUT KONTROL
        function komutKontrol(metin) {
            const altMetin = metin.toLowerCase();
            
            // TELEFON ARAMA
            if (altMetin.includes("ara")) {
                let numara = null;
                let arananKisi = "";
                
                for (const [isim, tel] of Object.entries(rehber)) {
                    if (altMetin.includes(isim)) {
                        numara = tel;
                        arananKisi = isim;
                        break;
                    }
                }
                
                if (!numara) {
                    const numaraMatch = metin.match(/\d{10,11}/);
                    if (numaraMatch) {
                        numara = "+90" + numaraMatch[0].replace(/^0/, '');
                    }
                }
                
                if (numara) {
                    const cevap = "Tamam Birtanem, " + (arananKisi || numara) + " arıyorum!";
                    ekranaYaz("asistan", cevap);
                    sesliOku(cevap);
                    
                    const aramaDiv = document.createElement("div");
                    aramaDiv.style.cssText = "margin: 15px 0; text-align: center;";
                    aramaDiv.innerHTML = '<a href="tel:' + numara + '" style="background: #25D366; color: white; padding: 15px 30px; border-radius: 10px; text-decoration: none; font-size: 20px; display: inline-block;">📞 ' + numara + ' Ara</a>';
                    document.getElementById("gecmis").appendChild(aramaDiv);
                    
                    return true;
                }
            }
            
            // WHATSAPP
            if (altMetin.includes("whatsapp")) {
                let numara = null;
                const numaraMatch = metin.match(/\d{10,11}/);
                if (numaraMatch) {
                    numara = "90" + numaraMatch[0].replace(/^0/, '');
                }
                
                if (numara) {
                    const url = "https://wa.me/" + numara;
                    window.open(url, '_blank');
                    const cevap = "WhatsApp açılıyor Birtanem!";
                    ekranaYaz("asistan", cevap);
                    sesliOku(cevap);
                    return true;
                }
            }
            
            // HATIRLATICI
            if (altMetin.includes("hatırlat")) {
                const dakikaMatch = altMetin.match(/(\d+)\s*dakika/);
                if (dakikaMatch) {
                    const dakika = parseInt(dakikaMatch[1]);
                    const mesaj = metin.replace(/.*hatırlat/i, '').trim() || "Hatırlatma zamanı!";
                    
                    setTimeout(() => {
                        alert("⏰ HATIRLATMA: " + mesaj);
                        sesliOku("Birtanem! " + mesaj);
                    }, dakika * 60 * 1000);
                    
                    const cevap = dakika + " dakika sonra hatırlatacağım!";
                    ekranaYaz("asistan", cevap);
                    sesliOku(cevap);
                    return true;
                }
            }
            
            // NOT ALMA
            if (altMetin.includes("not al")) {
                const notMetni = metin.replace(/not al[:\s]*/i, '').trim();
                if (notMetni) {
                    const notlar = JSON.parse(localStorage.getItem("dadasNotlar") || "[]");
                    notlar.push({ metin: notMetni, tarih: new Date().toLocaleString('tr-TR') });
                    localStorage.setItem("dadasNotlar", JSON.stringify(notlar));
                    const cevap = "Not aldım: " + notMetni;
                    ekranaYaz("asistan", cevap);
                    sesliOku(cevap);
                    return true;
                }
            }
            
            // NOTLARI OKU
            if (altMetin.includes("notlar") && altMetin.includes("oku")) {
                const notlar = JSON.parse(localStorage.getItem("dadasNotlar") || "[]");
                if (notlar.length === 0) {
                    const cevap = "Hiç notun yok.";
                    ekranaYaz("asistan", cevap);
                    sesliOku(cevap);
                } else {
                    let cevap = "Notların: ";
                    notlar.forEach((not, i) => {
                        cevap += (i+1) + ". " + not.metin + ". ";
                    });
                    ekranaYaz("asistan", cevap);
                    sesliOku(cevap);
                }
                return true;
            }
            
            // NOTLARI SİL
            if (altMetin.includes("notları sil")) {
                localStorage.removeItem("dadasNotlar");
                const cevap = "Notları sildim.";
                ekranaYaz("asistan", cevap);
                sesliOku(cevap);
                return true;
            }
            
            // YOUTUBE - VİDEO FİKRİ
            if (altMetin.includes("video fikri") || altMetin.includes("ne videosu")) {
                youtubeIstek("YouTube için yaratıcı video fikirleri ver. Kısa, net ve Türkçe cevap ver.");
                return true;
            }
            
            // YOUTUBE - BAŞLIK ÖNER
            if (altMetin.includes("başlık öner") || altMetin.includes("başlık ver")) {
                youtubeIstek("YouTube videosu için SEO uyumlu, dikkat çekici 5 başlık öner. Kısa ve Türkçe.");
                return true;
            }
            
            // YOUTUBE - AÇIKLAMA
            if (altMetin.includes("açıklama yaz") || altMetin.includes("açıklama ver")) {
                youtubeIstek("YouTube videosu için SEO uyumlu, ilgi çekici bir açıklama yaz. Kısa ve Türkçe.");
                return true;
            }
            
            // YOUTUBE - ETİKET
            if (altMetin.includes("etiket") || altMetin.includes("hashtag")) {
                youtubeIstek("YouTube videosu için popüler hashtag ve etiketler ver. 10-15 tane, Türkçe ve İngilizce karışık.");
                return true;
            }
            
            return false;
        }

        // YOUTUBE İSTEK
        async function youtubeIstek(prompt) {
            // API anahtarı kontrolü
            if (!API_KEY || API_KEY === "gsk_mWOzWQsfUCCaXYCswXA3WGdyb3FYKf8..." || API_KEY.includes("BURAYA")) {
                document.getElementById("durumMetni").innerText = "❌ API anahtarı eksik!";
                ekranaYaz("asistan", "API anahtarım eksik Birtanem! Lütfen ekle.");
                return;
            }
            
            // Mikrofonu KAPAT - asistan konuşuyor
            asistanKonuşuyor = true;
            if (recognition && dinlemeAktif) {
                try { recognition.stop(); } catch(e) {}
            }
            
            const sistemMesaji = {
                role: "system", 
                content: "Sen Dadaş'sın, Birtanem'in YouTube asistanısın. Kısa, samimi ve Türkçe cevap ver."
            };
            
            const userMesaj = { role: "user", content: prompt };
            const tumMesajlar = [sistemMesaji, userMesaj];
            
            try {
                const response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
                    method: "POST",
                    headers: {
                        "Authorization": "Bearer " + API_KEY,
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        model: "llama-3.3-70b-versatile",
                        messages: tumMesajlar
                    })
                });
                const data = await response.json();
                if (!response.ok) throw new Error(`API: ${data.error?.message || 'Hata'}`);
                if (!data.choices || data.choices.length === 0) throw new Error("Boş cevap");
                
                const cevap = data.choices[0].message.content;
                ekranaYaz("asistan", "🎬 " + cevap);
                sesliOku(cevap);
            } catch (error) {
                document.getElementById("durumMetni").innerText = "❌ " + error.message;
                ekranaYaz("asistan", "Hata: " + error.message);
            } finally {
                // İşlem bitince mikrofonu AÇ
                asistanKonuşuyor = false;
                if (dinlemeAktif) {
                    setTimeout(() => {
                        try { recognition.start(); } catch(e) {}
                    }, 500);
                }
            }
        }

        // YAPAY ZEKA
        async function yapayZekayaSor() {
            // API anahtarı kontrolü
            if (!API_KEY || API_KEY === "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" || API_KEY.includes("BURAYA")) {
                document.getElementById("durumMetni").innerText = "❌ API anahtarı eksik!";
                ekranaYaz("asistan", "API anahtarım eksik Birtanem! Lütfen ekle.");
                return;
            }
            
            // Mikrofonu KAPAT - asistan konuşuyor
            asistanKonuşuyor = true;
            if (recognition && dinlemeAktif) {
                try { recognition.stop(); } catch(e) {}
            }
            
            const sistemMesaji = {
                role: "system", 
                content: "Sen Dadaş'sın, Birtanem'in kişisel asistanısın. Kısa, samimi ve Türkçe cevap ver."
            };
            const tumMesajlar = [sistemMesaji, ...sohbetGecmisi];
            
            try {
                const response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
                    method: "POST",
                    headers: {
                        "Authorization": "Bearer " + API_KEY,
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        model: "llama-3.3-70b-versatile",
                        messages: tumMesajlar
                    })
                });
                const data = await response.json();
                if (!response.ok) throw new Error(`API: ${data.error?.message || 'Hata'}`);
                if (!data.choices || data.choices.length === 0) throw new Error("Boş cevap");
                
                const cevap = data.choices[0].message.content;
                ekranaYaz("asistan", cevap);
                sohbetGecmisi.push({role: "assistant", content: cevap});
                sesliOku(cevap);
            } catch (error) {
                document.getElementById("durumMetni").innerText = "❌ " + error.message;
                ekranaYaz("asistan", "Hata: " + error.message);
            } finally {
                // İşlem bitince mikrofonu AÇ
                asistanKonuşuyor = false;
                if (dinlemeAktif) {
                    setTimeout(() => {
                        try { recognition.start(); } catch(e) {}
                    }, 500);
                }
            }
        }

        // SESLİ OKUMA (KESİN ÇÖZÜM - Feedback loop tamamen önleniyor)
        function sesliOku(metin) {
            if ('speechSynthesis' in window) {
                // Mikrofonu KAPAT - asistan konuşuyor
                asistanKonuşuyor = true;
                if (recognition && dinlemeAktif) {
                    try { recognition.stop(); } catch(e) {}
                }
                
                // Önceki sesleri durdur
                window.speechSynthesis.cancel();
                
                const cumleler = metin.split(/(?<=[.!?])\s+/).filter(c => c.trim().length > 0);
                
                let cumleIndex = 0;
                
                function sonrakiCumleyiOku() {
                    if (cumleIndex >= cumleler.length) {
                        // Tüm cümleler bitti - mikrofonu AÇ
                        asistanKonuşuyor = false;
                        if (dinlemeAktif) {
                            setTimeout(() => {
                                try { recognition.start(); } catch(e) {}
                            }, 500);
                        }
                        return;
                    }
                    
                    const cumle = cumleler[cumleIndex];
                    const utterance = new SpeechSynthesisUtterance(cumle);
                    utterance.lang = 'tr-TR';
                    utterance.rate = 0.9;
                    utterance.pitch = 1.0;
                    utterance.volume = 1.0;
                    if (seciliSes) utterance.voice = seciliSes;
                    
                    utterance.onend = function() {
                        cumleIndex++;
                        sonrakiCumleyiOku();
                    };
                    
                    window.speechSynthesis.speak(utterance);
                }
                
                sonrakiCumleyiOku();
            }
        }

        function ekranaYaz(tip, metin) {
            const gecmisDiv = document.getElementById("gecmis");
            const mesajDiv = document.createElement("div");
            mesajDiv.className = "mesaj " + tip;
            mesajDiv.innerText = metin;
            gecmisDiv.appendChild(mesajDiv);
            gecmisDiv.scrollTop = gecmisDiv.scrollHeight;
        }
    </script>
</body>
</html>
