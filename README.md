        <div className="bg-gray-100 min-h-screen font-sans text-gray-800 flex flex-col items-center justify-center p-4">
            <Chatbot />
        </div>
    );
}

// --- Chatbot Component ---
function Chatbot() {
    const [messages, setMessages] = useState([]);
    const [input, setInput] = useState('');
    const [isLoading, setIsLoading] = useState(false);
    const [language, setLanguage] = useState('hi'); // Default to Hindi
    const messagesEndRef = useRef(null);

    const scrollToBottom = () => {
        messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
    };

    useEffect(scrollToBottom, [messages]);

    const addMessage = (text, sender, suggestions = null) => {
        setMessages(prev => [...prev, { text, sender, suggestions }]);
    };

    useEffect(() => {
        const welcomeMessage = language === 'en' 
            ? 'Hello, welcome to Pawanveg delivery service. How can we help you?'
            : 'नमस्ते, Pawanveg delivery service में आपका स्वागत है। हम आपकी क्या सहायता कर सकते हैं?';
        
        const suggestionOptions = language === 'en'
            ? ['Track Order', 'File a Complaint', 'See Offers', 'Give Suggestion']
            : ['Order Track Karein', 'Shikayat Darj Karein', 'Offers Dekhein', 'Sujhav Dein'];

        addMessage(welcomeMessage, 'bot', suggestionOptions);
    }, [language]);

    // --- Gemini API Call Function ---
    const getGeminiResponse = async (prompt, chatHistory) => {
        const apiKey = ""; // API key will be handled by the environment
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
        
        const fullHistory = [...chatHistory, { role: "user", parts: [{ text: prompt }] }];

        const payload = {
            contents: fullHistory,
        };

        try {
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            if (!response.ok) {
                throw new Error(`API request failed with status ${response.status}`);
            }
            const result = await response.json();
            if (result.candidates && result.candidates.length > 0) {
                return result.candidates[0].content.parts[0].text;
            }
            return language === 'en' ? "I couldn't process that. Please try again." : "मैं यह समझ नहीं पाया। कृपया दोबारा प्रयास करें।";
        } catch (error) {
            console.error("Gemini API Error:", error);
            return language === 'en' ? "Sorry, I'm having trouble connecting to my brain right now." : "क्षमा करें, मुझे अभी अपने सिस्टम से कनेक्ट करने में समस्या हो रही है।";
        }
    };

    const triggerBotResponse = async (userMessage) => {
        setIsLoading(true);

        const chatHistory = messages.map(msg => ({
            role: msg.sender === 'user' ? 'user' : 'model',
            parts: [{ text: msg.text }]
        }));

        // --- The New, More Realistic "Master Prompt" ---
        const mainPrompt = `
            You are a very friendly, cheerful, and helpful AI assistant for 'PawanVeg' (pawanveg.com), a food delivery service.
            The user is speaking ${language === 'en' ? 'English' : 'Hindi'}. Always respond in their language with a warm and positive tone.

            **Your Capabilities:**
            - **Order Tracking:** If the user wants to track an order, ask them for their order ID.
            - **Complaints:** If the user has a complaint, be very empathetic. Apologize for the trouble and ask for their order ID and the specific problem.
            - **Offers:** If asked about offers, mention that all offers are on the PawanVeg app and give an example like "a 'BUY ONE GET ONE' offer on pizzas this week. �".
            - **Suggestions:** If the user wants to give a suggestion, thank them enthusiastically and encourage them to share it.
            - **Recipes:** If the user asks for a recipe, first ask what recipe they are looking for. If they provide an ingredient, give a simple recipe for it.
            - **General Chat:** If the user just wants to chat about food, engage in a friendly conversation. Ask follow-up questions.

            **IMPORTANT RULE**: Your expertise is ONLY food, recipes, and food delivery. If the user asks about anything else (like politics, movies, etc.), you MUST politely refuse. Say something like "Mera kaam food se related hai, is vishay par main aapki madad nahi kar paunga, maaf karein 🙏" in Hindi, or "My expertise is all about food, so I can't help with that, sorry! 🙏" in English.

            **User's current message is**: "${userMessage}"

            Based on all these instructions, generate a direct, natural, and helpful response.
        `;

        // The new logic is much simpler: just one API call.
        const botResponseText = await getGeminiResponse(mainPrompt, chatHistory);

        addMessage(botResponseText, 'bot');
        setIsLoading(false);
    };

    const handleSend = async () => {
        if (!input.trim() || isLoading) return;
        const userMessage = input;
        addMessage(userMessage, 'user');
        setInput('');
        triggerBotResponse(userMessage);
    };

    const handleSuggestionClick = (suggestion) => {
        addMessage(suggestion, 'user');
        setMessages(prev => prev.map(m => ({ ...m, suggestions: null })));
        triggerBotResponse(suggestion);
    };

    return (
        <div className="w-full max-w-lg h-[80vh] bg-white rounded-2xl shadow-2xl flex flex-col">
            <div className="flex justify-between items-center p-4 bg-red-500 text-white rounded-t-2xl">
                <div className="flex items-center gap-3">
                    <FoodIcon />
                    <h3 className="font-bold text-lg">Pawanveg Support</h3>
                </div>
                 <div className="flex items-center gap-2">
                    <select value={language} onChange={e => setLanguage(e.target.value)} className="bg-red-600 text-white text-sm rounded p-1 border-0 focus:ring-0 cursor-pointer">
                        <option value="en">EN</option>
                        <option value="hi">HI</option>
                    </select>
                </div>
            </div>
            <div className="flex-grow p-4 overflow-y-auto bg-gray-50">
                {messages.map((msg, index) => (
                    <div key={index} className="mb-3">
                        <div className={`flex ${msg.sender === 'user' ? 'justify-end' : 'justify-start'}`}>
                            <div className={`max-w-[80%] py-2 px-3 rounded-lg ${msg.sender === 'user' ? 'bg-yellow-300 text-gray-800' : 'bg-white text-gray-800 border'}`}>
                                {msg.text.split('\n').map((line, i) => <p key={i}>{line}</p>)}
                            </div>
                        </div>
                        {msg.suggestions && (
                            <div className="flex flex-wrap gap-2 mt-2 justify-start">
                                {msg.suggestions.map((suggestion, i) => (
                                    <button
                                        key={i}
                                        onClick={() => handleSuggestionClick(suggestion)}
                                        className="bg-white border border-red-500 text-red-500 text-sm px-3 py-1 rounded-full hover:bg-red-100 transition-colors"
                                    >
                                        {suggestion}
                                    </button>
                                ))}
                            </div>
                        )}
                    </div>
                ))}
                {isLoading && (
                    <div className="flex justify-start mb-3">
                         <div className="p-2 rounded-lg bg-white border">
                            <div className="flex items-center gap-2 p-1">
                                <div className="w-2 h-2 bg-gray-400 rounded-full animate-pulse"></div>
                                <div className="w-2 h-2 bg-gray-400 rounded-full animate-pulse [animation-delay:0.2s]"></div>
                                <div className="w-2 h-2 bg-gray-400 rounded-full animate-pulse [animation-delay:0.4s]"></div>
                            </div>
                         </div>
                    </div>
                )}
                <div ref={messagesEndRef} />
            </div>
            <div className="p-3 border-t flex items-center gap-2 bg-white rounded-b-2xl">
                <div className="relative flex-grow">
                     <input
                        type="text"
                        value={input}
                        onChange={(e) => setInput(e.target.value)}
                        onKeyPress={(e) => e.key === 'Enter' && handleSend()}
                        placeholder={language === 'en' ? 'Type your query...' : 'Apna sawaal likhein...'}
                        className="w-full p-3 pr-10 border rounded-full focus:outline-none focus:ring-2 focus:ring-red-300"
                    />
                    <span className="absolute right-4 top-1/2 -translate-y-1/2 text-yellow-500">
                        <SparklesIcon />
                    </span>
                </div>
                <button onClick={handleSend} className="bg-red-500 text-white p-3 rounded-full hover:bg-red-600 disabled:bg-gray-400 transition-colors" disabled={isLoading}>
                    <SendIcon />
                </button>
            </div>
        </div>
    );
}
�
