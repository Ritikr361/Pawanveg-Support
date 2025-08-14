<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pawanveg Support Chatbot</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwin dcss.com"></script>
    <!-- React Libraries -->
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
    <!-- Babel to translate JSX in the browser -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body class="bg-gray-100">

    <div id="root"></div>

    <script type="text/babel">
        // --- Helper Icons (as SVGs) ---
        const SendIcon = () => (
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="lucide lucide-send">
            <path d="m22 2-7 20-4-9-9-4Z" />
            <path d="M22 2 11 13" />
          </svg>
        );

        const FoodIcon = () => (
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M12 22c5.523 0 10-4.477 10-10H2c0 5.523 4.477 10 10 10z"/><path d="M16 12c0-2.21-1.79-4-4-4s-4 1.79-4 4"/><path d="M18 8c0-1.105-.895-2-2-2s-2 .895-2 2"/><path d="M10 8c0-1.105-.895-2-2-2s-2 .895-2 2"/></svg>
        );

        const WhatsAppIcon = () => (
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16">
                <path d="M13.601 2.326A7.85 7.85 0 0 0 7.994 0C3.627 0 .068 3.558.064 7.926c0 1.399.366 2.76 1.057 3.965L0 16l4.204-1.102a7.9 7.9 0 0 0 3.79.965h.004c4.368 0 7.926-3.558 7.93-7.93A7.9 7.9 0 0 0 13.6 2.326zM7.994 14.521a6.6 6.6 0 0 1-3.356-.92l-.24-.144-2.494.654.666-2.433-.156-.251a6.56 6.56 0 0 1-1.007-3.505c0-3.626 2.957-6.584 6.591-6.584a6.56 6.56 0 0 1 4.66 1.931 6.56 6.56 0 0 1 1.928 4.66c-.004 3.639-2.961 6.592-6.592 6.592m3.615-4.934c-.197-.099-1.17-.578-1.353-.646-.182-.065-.315-.099-.445.099-.133.197-.513.646-.627.775-.114.133-.232.148-.43.05-.197-.1-.836-.308-1.592-.985-.59-.525-.985-1.175-1.103-1.372-.114-.198-.011-.304.088-.403.087-.088.197-.232.296-.346.1-.114.133-.198.198-.33.065-.134.034-.248-.015-.347-.05-.1-.445-1.076-.612-1.47-.16-.389-.323-.335-.445-.34-.114-.007-.247-.007-.38-.007a.73.73 0 0 0-.529.247c-.182.198-.691.677-.691 1.654s.71 1.916.81 2.049c.098.133 1.394 2.132 3.383 2.992.47.205.84.326 1.129.418.475.152.904.129 1.246.08.38-.058 1.171-.48 1.338-.943.164-.464.164-.86.114-.943-.049-.084-.182-.133-.38-.232"/>
            </svg>
        );

        // --- Main App Component ---
        function App() {
            return (
                <div className="bg-gray-100 min-h-screen font-sans text-gray-800 flex flex-col items-center justify-center p-4">
                    <Chatbot />
                </div>
            );
        }

        // --- Chatbot Component ---
        function Chatbot() {
            const [messages, setMessages] = React.useState([]);
            const [input, setInput] = React.useState('');
            const [isLoading, setIsLoading] = React.useState(false);
            const [language, setLanguage] = React.useState('hi'); // Default to Hindi
            const [conversationState, setConversationState] = React.useState('idle'); // idle, gathering_complaint
            const [complaintDetails, setComplaintDetails] = React.useState('');
            const messagesEndRef = React.useRef(null);

            const scrollToBottom = () => {
                messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
            };

            React.useEffect(scrollToBottom, [messages]);

            const addMessage = (text, sender, suggestions = null) => {
                setMessages(prev => [...prev, { text, sender, suggestions }]);
            };

            React.useEffect(() => {
                const welcomeMessage = language === 'en' 
                    ? 'Hello, welcome to Pawanveg delivery service. How can we help you?'
                    : 'नमस्ते, Pawanveg delivery service में आपका स्वागत है। हम आपकी क्या सहायता कर सकते हैं?';
                
                const suggestionOptions = language === 'en'
                    ? ['Track Order', 'File a Complaint', 'See Offers', 'Give Suggestion']
                    : ['Order Track Karein', 'Shikayat Darj Karein', 'Offers Dekhein', 'Sujhav Dein'];

                addMessage(welcomeMessage, 'bot', suggestionOptions);
            }, [language]);

            const resetConversation = () => {
                setConversationState('idle');
                setComplaintDetails('');
            };

            const triggerBotResponse = async (userMessage) => {
                setIsLoading(true);
                await new Promise(resolve => setTimeout(resolve, 500));

                let botResponseText = '';
                const lowerCaseMessage = userMessage.toLowerCase();

                // State-based logic for handling complaints
                if (conversationState === 'gathering_complaint') {
                    setComplaintDetails(userMessage);
                    botResponseText = language === 'en'
                        ? `Thank you for the details. Should I send this information to our support team on WhatsApp?`
                        : `जानकारी के लिए धन्यवाद। क्या मैं यह जानकारी हमारी सपोर्ट टीम को WhatsApp पर भेज दूँ?`;
                    
                    const confirmationOptions = language === 'en' ? ['Yes, send it', 'No, cancel'] : ['हाँ, भेजें', 'नहीं, रहने दें'];
                    addMessage(botResponseText, 'bot', confirmationOptions);
                    setConversationState('confirm_whatsapp'); // Move to next state
                    setIsLoading(false);
                    return;
                }

                if (conversationState === 'confirm_whatsapp') {
                    if (lowerCaseMessage.includes('yes') || lowerCaseMessage.includes('हाँ')) {
                        const whatsappNumber = '918989161188';
                        const messageText = encodeURIComponent(`Pawanveg Support Request:\n\n*Complaint Details:*\n${complaintDetails}`);
                        const whatsappUrl = `https://wa.me/${whatsappNumber}?text=${messageText}`;
                        
                        botResponseText = language === 'en'
                            ? 'Great! Please click the button below to send your complaint to our support team. They will get back to you shortly.'
                            : 'ठीक है! कृपया नीचे दिए गए बटन पर क्लिक करके अपनी शिकायत हमारी सपोर्ट टीम को भेजें। वे जल्द ही आपसे संपर्क करेंगे।';
                        
                        addMessage(botResponseText, 'bot', [{ type: 'whatsapp', url: whatsappUrl, text: 'Send on WhatsApp' }]);
                    } else {
                        botResponseText = language === 'en'
                            ? 'Okay, I have cancelled the request. Is there anything else I can help you with?'
                            : 'ठीक है, मैंने अनुरोध रद्द कर दिया है। क्या मैं आपकी कोई और मदद कर सकता हूँ?';
                        addMessage(botResponseText, 'bot');
                    }
                    resetConversation();
                    setIsLoading(false);
                    return;
                }

                // Standard logic for other queries
                if (lowerCaseMessage.includes('track')) {
                    botResponseText = language === 'en' ? "Sure, please provide your order ID." : "ज़रूर, कृपया अपना ऑर्डर ID बताएं।";
                } else if (lowerCaseMessage.includes('shikayat') || lowerCaseMessage.includes('complaint')) {
                    botResponseText = language === 'en' ? "I'm sorry to hear that. Please describe your issue in detail." : "हमें सुनकर दुख हुआ। कृपया अपनी समस्या विस्तार से बताएं।";
                    setConversationState('gathering_complaint');
                } else if (lowerCaseMessage.includes('offer')) {
                    botResponseText = language === 'en' ? "Currently, we have a 'BUY ONE GET ONE' offer on all large pizzas! 🍕" : "अभी हमारे सभी बड़े पिज़्ज़ा पर 'एक खरीदें, एक मुफ़्त पाएं' का ऑफ़र चल रहा है! 🍕";
                } else if (lowerCaseMessage.includes('sujhav') || lowerCaseMessage.includes('suggestion')) {
                    botResponseText = language === 'en' ? "We'd love to hear your suggestion! Please let us know what's on your mind." : "हमें आपका सुझाव सुनकर बहुत खुशी होगी! कृपया बताएं कि आपके मन में क्या है।";
                } else {
                    botResponseText = language === 'en' ? "I can help with tracking orders, complaints, offers, and suggestions." : "मैं ऑर्डर ट्रैक करने, शिकायत दर्ज करने, ऑफ़र जानने और सुझाव देने में आपकी मदद कर सकता हूँ।";
                }

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
                                        {msg.suggestions.map((suggestion, i) => {
                                            if (suggestion.type === 'whatsapp') {
                                                return (
                                                    <a
                                                        key={i}
                                                        href={suggestion.url}
                                                        target="_blank"
                                                        rel="noopener noreferrer"
                                                        className="bg-green-500 text-white font-bold text-sm px-4 py-2 rounded-full hover:bg-green-600 transition-colors inline-flex items-center gap-2"
                                                    >
                                                        <WhatsAppIcon /> {suggestion.text}
                                                    </a>
                                                )
                                            }
                                            return (
                                            <button
                                                key={i}
                                                onClick={() => handleSuggestionClick(suggestion)}
                                                className="bg-white border border-red-500 text-red-500 text-sm px-3 py-1 rounded-full hover:bg-red-100 transition-colors"
                                            >
                                                {suggestion}
                                            </button>
                                        )})}
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
                                className="w-full p-3 border rounded-full focus:outline-none focus:ring-2 focus:ring-red-300"
                            />
                        </div>
                        <button onClick={handleSend} className="bg-red-500 text-white p-3 rounded-full hover:bg-red-600 disabled:bg-gray-400 transition-colors" disabled={isLoading}>
                            <SendIcon />
                        </button>
                    </div>
                </div>
            );
        }

        ReactDOM.render(<App />, document.getElementById('root'));
    </script>

</body>
</html>
