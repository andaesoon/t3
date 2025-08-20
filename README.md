# t3
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>사진 번역기 웹 앱</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;800&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Custom scrollbar style for the result box */
        .result-box::-webkit-scrollbar {
            width: 8px;
        }
        .result-box::-webkit-scrollbar-thumb {
            background-color: #cbd5e1;
            border-radius: 4px;
        }
        .result-box::-webkit-scrollbar-track {
            background-color: #f1f5f9;
        }
    </style>
</head>
<body class="bg-gray-100 p-4">

    <div class="min-h-screen flex flex-col items-center justify-center">
        <div class="bg-white rounded-xl shadow-lg p-6 sm:p-8 w-full max-w-4xl text-center">
            <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-900 mb-2">
                사진 번역기 웹 앱
            </h1>
            <p class="text-sm sm:text-base text-gray-600 mb-6">
                사진 속 텍스트를 원하는 언어로 번역합니다.
            </p>

            <div id="disclaimer" class="bg-yellow-100 border border-yellow-300 text-yellow-800 p-4 rounded-xl mb-6 text-sm">
                <h3 class="font-bold mb-1">참고 사항</h3>
                <p>
                    손글씨, 특수 글꼴 또는 저화질 이미지의 경우 OCR 정확도가 떨어질 수 있습니다. 또한, 이 앱은 번역된 텍스트를 원본 이미지 위에 겹쳐 표시하는 기능은 제공하지 않으며, 번역된 텍스트만 별도로 추출합니다.
                </p>
                <button onclick="document.getElementById('disclaimer').style.display = 'none';" class="mt-2 text-yellow-600 underline text-xs">
                    닫기
                </button>
            </div>

            <div class="mb-6 flex flex-col sm:flex-row items-center justify-center space-y-4 sm:space-y-0 sm:space-x-4">
                <label class="w-full sm:w-auto bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-full cursor-pointer transition duration-300 ease-in-out transform hover:scale-105 shadow-md">
                    사진 업로드
                    <input
                        id="file-input"
                        type="file"
                        accept="image/jpeg, image/png"
                        class="hidden"
                    />
                </label>
                <div class="relative w-full sm:w-auto">
                    <select id="language-select" class="block w-full sm:w-48 bg-gray-200 border border-gray-300 text-gray-700 py-2 px-4 pr-8 rounded-full leading-tight focus:outline-none focus:bg-white focus:border-gray-500">
                        <option value="ko_to_zh">한국어 → 중국어</option>
                        <option value="ko_to_ru">한국어 → 러시아어</option>
                    </select>
                    <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
                        <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"/></svg>
                    </div>
                </div>
                <button
                    id="translate-button"
                    class="w-full sm:w-auto font-bold py-2 px-6 rounded-full transition duration-300 ease-in-out transform hover:scale-105 shadow-md bg-gray-400 text-gray-600 cursor-not-allowed"
                    disabled
                >
                    번역하기
                </button>
            </div>

            <div id="error-message" class="hidden bg-red-100 border border-red-400 text-red-700 p-3 rounded-lg mb-4 text-sm"></div>

            <div id="result-container" class="hidden flex flex-col md:flex-row items-center md:items-start justify-center space-y-6 md:space-y-0 md:space-x-8">
                <div class="w-full md:w-1/2 flex-shrink-0">
                    <h3 class="text-xl font-bold mb-2">원본 이미지</h3>
                    <img id="image-preview" src="#" alt="업로드된 이미지" class="w-full h-auto rounded-lg shadow-md border-2 border-gray-200" />
                </div>
                <div class="w-full md:w-1/2 flex-grow">
                    <h3 class="text-xl font-bold mb-2">번역 결과</h3>
                    <div class="relative bg-gray-50 border border-gray-300 rounded-lg p-4 h-64 overflow-y-auto text-left shadow-inner result-box">
                        <div id="loading-spinner" class="absolute inset-0 hidden items-center justify-center bg-white bg-opacity-75">
                            <svg class="animate-spin h-8 w-8 text-blue-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                            </svg>
                        </div>
                        <p id="translated-text" class="whitespace-pre-wrap text-gray-800"></p>
                        <p id="placeholder-text" class="text-gray-400 italic">번역 결과를 보려면 '번역하기' 버튼을 누르세요.</p>
                    </div>
                    <div id="action-buttons" class="hidden justify-start space-x-4 mt-4">
                        <button
                            id="copy-text-button"
                            class="bg-purple-600 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded-full transition duration-300 shadow-md transform hover:scale-105"
                        >
                            텍스트 복사
                        </button>
                        <button
                            id="download-image-button"
                            class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-full transition duration-300 shadow-md transform hover:scale-105"
                        >
                            원본 이미지 다운로드
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const fileInput = document.getElementById('file-input');
            const translateButton = document.getElementById('translate-button');
            const imagePreview = document.getElementById('image-preview');
            const translatedTextEl = document.getElementById('translated-text');
            const errorMessageEl = document.getElementById('error-message');
            const resultContainer = document.getElementById('result-container');
            const loadingSpinner = document.getElementById('loading-spinner');
            const placeholderText = document.getElementById('placeholder-text');
            const actionButtons = document.getElementById('action-buttons');
            const copyTextButton = document.getElementById('copy-text-button');
            const downloadImageButton = document.getElementById('download-image-button');
            const languageSelect = document.getElementById('language-select');
            let selectedFile = null;

            // Helper function to convert file to Base64
            const fileToBase64 = (file) => {
                return new Promise((resolve, reject) => {
                    const reader = new FileReader();
                    reader.readAsDataURL(file);
                    reader.onload = () => resolve(reader.result.split(',')[1]);
                    reader.onerror = (error) => reject(error);
                });
            };

            // Image upload handler
            fileInput.addEventListener('change', (event) => {
                const file = event.target.files[0];
                if (file) {
                    if (file.type !== 'image/jpeg' && file.type !== 'image/png') {
                        showError('JPG 또는 PNG 파일만 업로드할 수 있습니다.');
                        resetState();
                        return;
                    }
                    selectedFile = file;
                    imagePreview.src = URL.createObjectURL(file);
                    translatedTextEl.textContent = '';
                    placeholderText.style.display = 'block';
                    actionButtons.classList.add('hidden');
                    translateButton.disabled = false;
                    translateButton.classList.remove('bg-gray-400', 'text-gray-600', 'cursor-not-allowed');
                    translateButton.classList.add('bg-green-600', 'hover:bg-green-700', 'text-white');
                    resultContainer.classList.remove('hidden');
                    hideError();
                } else {
                    resetState();
                }
            });

            // Translate button handler
            translateButton.addEventListener('click', async () => {
                if (!selectedFile) {
                    showError('이미지 파일을 먼저 업로드해주세요.');
                    return;
                }

                setLoading(true);
                hideError();

                try {
                    const base64Image = await fileToBase64(selectedFile);
                    const selectedLanguage = languageSelect.value;
                    let prompt;

                    if (selectedLanguage === 'ko_to_zh') {
                        prompt = "Analyze the text in this image. The text is in Korean. Translate all detected text into Chinese and return only the translated Chinese text.";
                    } else if (selectedLanguage === 'ko_to_ru') {
                        prompt = "Analyze the text in this image. The text is in Korean. Translate all detected text into Russian and return only the translated Russian text.";
                    }

                    const payload = {
                        contents: [
                            {
                                role: "user",
                                parts: [
                                    { text: prompt },
                                    {
                                        inlineData: {
                                            mimeType: selectedFile.type,
                                            data: base64Image
                                        }
                                    }
                                ]
                            }
                        ],
                    };
                    
                    const apiKey = "AIzaSyCWpt1uxNSdV2B_QKV5B1G7Vd6osT3xCIc"; // 요청하신 API 키를 적용했습니다.
                    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

                    const apiResponse = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!apiResponse.ok) {
                        throw new Error('API 호출 중 오류가 발생했습니다.');
                    }
                    
                    const result = await apiResponse.json();
                    const text = result?.candidates?.[0]?.content?.parts?.[0]?.text || '번역된 텍스트를 찾을 수 없습니다.';
                    translatedTextEl.textContent = text;
                    
                    if (text.length > 0 && text !== '번역된 텍스트를 찾을 수 없습니다.') {
                        actionButtons.classList.remove('hidden');
                    } else {
                        actionButtons.classList.add('hidden');
                    }

                } catch (err) {
                    console.error(err);
                    showError('번역에 실패했습니다. 네트워크 연결을 확인하고 다시 시도해주세요.');
                    translatedTextEl.textContent = '';
                    actionButtons.classList.add('hidden');
                } finally {
                    setLoading(false);
                }
            });

            // Copy text button handler
            copyTextButton.addEventListener('click', () => {
                if (translatedTextEl.textContent) {
                    document.execCommand('copy');
                    // Use a custom modal or message box instead of alert()
                    const modal = document.createElement('div');
                    modal.textContent = '번역된 텍스트가 클립보드에 복사되었습니다.';
                    modal.className = 'fixed top-10 left-1/2 -translate-x-1/2 bg-gray-800 text-white px-4 py-2 rounded-lg shadow-lg transition-opacity duration-300';
                    document.body.appendChild(modal);
                    setTimeout(() => {
                        modal.style.opacity = '0';
                        setTimeout(() => modal.remove(), 300);
                    }, 2000);
                }
            });
            
            // Clipboard copy fallback
            document.addEventListener('copy', function(e) {
                if (translatedTextEl.textContent) {
                    e.clipboardData.setData('text/plain', translatedTextEl.textContent);
                    e.preventDefault();
                }
            });

            // Download image button handler
            downloadImageButton.addEventListener('click', () => {
                if (selectedFile) {
                    const link = document.createElement('a');
                    link.href = URL.createObjectURL(selectedFile);
                    link.download = 'translated_image.jpg';
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);
                }
            });

            // UI state management functions
            const setLoading = (isLoading) => {
                translateButton.disabled = isLoading;
                if (isLoading) {
                    loadingSpinner.classList.remove('hidden');
                    loadingSpinner.classList.add('flex');
                    translateButton.textContent = '번역 중...';
                    placeholderText.style.display = 'none';
                } else {
                    loadingSpinner.classList.add('hidden');
                    loadingSpinner.classList.remove('flex');
                    translateButton.textContent = '번역하기';
                }
            };
            
            const showError = (message) => {
                errorMessageEl.textContent = message;
                errorMessageEl.classList.remove('hidden');
            };
            
            const hideError = () => {
                errorMessageEl.classList.add('hidden');
            };
            
            const resetState = () => {
                selectedFile = null;
                imagePreview.src = '#';
                translatedTextEl.textContent = '';
                translateButton.disabled = true;
                translateButton.classList.remove('bg-green-600', 'hover:bg-green-700', 'text-white');
                translateButton.classList.add('bg-gray-400', 'text-gray-600', 'cursor-not-allowed');
                resultContainer.classList.add('hidden');
                hideError();
                placeholderText.style.display = 'block';
                actionButtons.classList.add('hidden');
            };
        });
    </script>
</body>
</html>
