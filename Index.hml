import React, { useState, useEffect, useMemo, useCallback } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, doc, setDoc, onSnapshot, collection, query, orderBy } from 'firebase/firestore';

// Global variables provided by the environment
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : '';

const App = () => {
  // Application state
  const [db, setDb] = useState(null);
  const [auth, setAuth] = useState(null);
  const [userId, setUserId] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [expenses, setExpenses] = useState([]);
  const [newExpense, setNewExpense] = useState({ description: '', amount: '', payerId: '', splitType: 'even' });
  const [salaries, setSalaries] = useState({ mySalary: '', partnerSalary: '' });
  const [isLoading, setIsLoading] = useState(true);
  const [statusMessage, setStatusMessage] = useState('');
  const [isDarkMode, setIsDarkMode] = useState(true);
  const [showExplanation, setShowExplanation] = useState(false);
  
  // User mapping
  const userName = "Karl";
  const partnerName = "Emma";
  const partnerId = "partner-emma";

  // Initialize Firebase and set up auth listener
  useEffect(() => {
    try {
      if (Object.keys(firebaseConfig).length > 0) {
        const app = initializeApp(firebaseConfig);
        const authInstance = getAuth(app);
        const dbInstance = getFirestore(app);
        setAuth(authInstance);
        setDb(dbInstance);

        const unsubscribeAuth = onAuthStateChanged(authInstance, async (user) => {
          if (user) {
            setUserId(user.uid);
            setIsAuthenticated(true);
            setIsLoading(false);
            console.log("Authenticated as:", user.uid);
          } else {
            console.log("No user signed in. Attempting anonymous sign-in or custom token.");
            try {
              if (initialAuthToken) {
                await signInWithCustomToken(authInstance, initialAuthToken);
              } else {
                await signInAnonymously(authInstance);
              }
            } catch (error) {
              console.error("Authentication failed:", error);
              setIsLoading(false);
              setStatusMessage("Kunde inte logga in. Kontrollera dina inställningar.");
            }
          }
        });

        return () => unsubscribeAuth();
      } else {
        setStatusMessage("Firebase-konfiguration saknas. Appen kan inte spara data.");
      }
    } catch (error) {
      console.error("Failed to initialize Firebase:", error);
      setStatusMessage("Ett fel uppstod vid start av appen.");
    }
  }, []);

  // Fetch expenses from Firestore in real-time
  useEffect(() => {
    if (db && userId) {
      const collectionPath = `/artifacts/${appId}/public/data/expenses`;
      const q = query(collection(db, collectionPath));

      const unsubscribe = onSnapshot(q, (querySnapshot) => {
        const expensesData = [];
        querySnapshot.forEach((doc) => {
          expensesData.push({ id: doc.id, ...doc.data() });
        });
        setExpenses(expensesData.sort((a, b) => b.timestamp - a.timestamp));
        console.log("Expenses updated.");
      }, (error) => {
        console.error("Failed to fetch expenses:", error);
        setStatusMessage("Kunde inte hämta kostnader.");
      });

      return () => unsubscribe();
    }
  }, [db, userId]);

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setNewExpense(prev => ({ ...prev, [name]: value }));
  };

  const handleSalaryChange = (e) => {
    const { name, value } = e.target;
    setSalaries(prev => ({ ...prev, [name]: value }));
  };

  const handleAddExpense = useCallback(async (e) => {
    e.preventDefault();
    if (!db || !userId) {
      setStatusMessage("Appen är inte redo. Vänligen vänta.");
      return;
    }
    const { description, amount, payerId, splitType } = newExpense;
    if (!description || !amount || !payerId || !splitType) {
      setStatusMessage("Alla fält måste fyllas i.");
      return;
    }
    const parsedAmount = parseFloat(amount);
    if (isNaN(parsedAmount) || parsedAmount <= 0) {
      setStatusMessage("Beloppet måste vara ett positivt nummer.");
      return;
    }
    
    if (splitType === 'salary' && (!salaries.mySalary || !salaries.partnerSalary)) {
        setStatusMessage("Du måste ange båda lönerna för att dela baserat på nettolön.");
        return;
    }

    try {
      const newDocRef = doc(collection(db, `/artifacts/${appId}/public/data/expenses`));
      await setDoc(newDocRef, {
        description: description,
        amount: parsedAmount,
        payerId: payerId,
        splitType: splitType,
        timestamp: Date.now(),
      });
      setNewExpense({ description: '', amount: '', payerId: '', splitType: 'even' });
      setStatusMessage("Kostnad tillagd!");
    } catch (error) {
      console.error("Failed to add expense:", error);
      setStatusMessage("Kunde inte lägga till kostnad.");
    }
  }, [db, userId, newExpense, salaries]);

  // Calculate the balance based on all expenses
  const balanceSummary = useMemo(() => {
    if (expenses.length === 0 || !salaries.mySalary || !salaries.partnerSalary) {
      return null;
    }

    const myId = userId;
    const totalSalary = parseFloat(salaries.mySalary) + parseFloat(salaries.partnerSalary);
    if (totalSalary <= 0) return null;

    const mySalaryRatio = parseFloat(salaries.mySalary) / totalSalary;
    const partnerSalaryRatio = parseFloat(salaries.partnerSalary) / totalSalary;

    const payerTotals = {};
    const effectiveBalances = {};
    [myId, partnerId].forEach(id => {
      payerTotals[id] = 0;
      effectiveBalances[id] = 0;
    });
    
    expenses.forEach(exp => {
      payerTotals[exp.payerId] = (payerTotals[exp.payerId] || 0) + exp.amount;
      
      let myPortion = 0;
      let partnerPortion = 0;

      if (exp.splitType === 'even') {
          myPortion = exp.amount / 2;
          partnerPortion = exp.amount / 2;
      } else if (exp.splitType === 'salary') {
          myPortion = exp.amount * mySalaryRatio;
          partnerPortion = exp.amount * partnerSalaryRatio;
      }
      
      if (exp.payerId === myId) {
          effectiveBalances[myId] += (exp.amount - myPortion);
          effectiveBalances[partnerId] -= partnerPortion;
      } else {
          effectiveBalances[myId] -= myPortion;
          effectiveBalances[partnerId] += (exp.amount - partnerPortion);
      }
    });

    const summary = {
      payerTotals: payerTotals,
      balances: effectiveBalances,
      whoOwesWhom: {},
    };

    const payers = Object.entries(effectiveBalances).filter(([, balance]) => balance > 0).sort((a, b) => b[1] - a[1]);
    const receivers = Object.entries(effectiveBalances).filter(([, balance]) => balance < 0).sort((a, b) => a[1] - b[1]);

    let i = 0, j = 0;
    while (i < payers.length && j < receivers.length) {
      const [payerId, payerBalance] = payers[i];
      const [receiverId, receiverBalance] = receivers[j];
      const amountToTransfer = Math.min(payerBalance, Math.abs(receiverBalance));

      summary.whoOwesWhom[receiverId] = summary.whoOwesWhom[receiverId] || {};
      summary.whoOwesWhom[receiverId][payerId] = (summary.whoOwesWhom[receiverId][payerId] || 0) + amountToTransfer;

      payers[i][1] -= amountToTransfer;
      receivers[j][1] += amountToTransfer;

      if (payers[i][1] < 0.01) i++;
      if (receivers[j][1] > -0.01) j++;
    }

    return summary;
  }, [expenses, salaries.mySalary, salaries.partnerSalary, userId]);

  const toggleTheme = () => {
    setIsDarkMode(prev => !prev);
  };

  if (isLoading) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-gray-50 dark:bg-gray-900 transition-colors duration-500">
        <div className="text-gray-900 dark:text-gray-100">
          <svg className="animate-spin h-10 w-10 text-teal-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <p className="mt-4 text-center">Laddar...</p>
        </div>
      </div>
    );
  }

  const getPayerName = (id) => id === userId ? userName : partnerName;

  return (
    <div className={`${isDarkMode ? 'dark bg-gray-900 text-white' : 'bg-gray-50 text-gray-900'} min-h-screen p-4 sm:p-8 font-sans transition-colors duration-500`}>
      <div className="container mx-auto max-w-2xl">
        <header className="flex justify-between items-center mb-6">
          <h1 className="text-3xl sm:text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-teal-400 to-emerald-500">
            Följ Pengarna
          </h1>
          <button
            onClick={toggleTheme}
            className="p-2 rounded-full bg-gray-200 dark:bg-gray-800 text-gray-800 dark:text-gray-200 transition-colors duration-300 shadow-md hover:scale-105"
            aria-label="Växla tema"
          >
            {isDarkMode ? '☀️' : '🌙'}
          </button>
        </header>

        <p className="text-center text-gray-600 dark:text-gray-400 mb-6">
          Enkel kostnadsdelning för er som delar allt.
        </p>

        {statusMessage && (
          <div className="bg-teal-100 dark:bg-teal-900 text-teal-700 dark:text-teal-200 p-3 rounded-md mb-6 shadow-sm text-center">
            {statusMessage}
          </div>
        )}
        
        <div className="p-4 sm:p-6 bg-white dark:bg-gray-800 rounded-3xl shadow-xl mb-6">
          <h2 className="text-xl sm:text-2xl font-semibold mb-4 text-gray-800 dark:text-white">Ange nettolöner</h2>
          <div className="flex flex-col sm:flex-row space-y-4 sm:space-y-0 sm:space-x-4 mb-4">
            <input
              type="number"
              name="mySalary"
              placeholder={`${userName}s nettolön`}
              value={salaries.mySalary}
              onChange={handleSalaryChange}
              className="w-full p-3 rounded-xl border border-gray-200 dark:border-gray-700 dark:bg-gray-700 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-300"
            />
            <input
              type="number"
              name="partnerSalary"
              placeholder={`${partnerName}s nettolön`}
              value={salaries.partnerSalary}
              onChange={handleSalaryChange}
              className="w-full p-3 rounded-xl border border-gray-200 dark:border-gray-700 dark:bg-gray-700 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-300"
            />
          </div>
        </div>

        <div className="p-4 sm:p-6 bg-white dark:bg-gray-800 rounded-3xl shadow-xl mb-6">
          <h2 className="text-xl sm:text-2xl font-semibold mb-4 text-gray-800 dark:text-white">Lägg till ny kostnad</h2>
          <form onSubmit={handleAddExpense} className="flex flex-col space-y-4">
            <input
              type="text"
              name="description"
              placeholder="Beskrivning (t.ex. Hyra)"
              value={newExpense.description}
              onChange={handleInputChange}
              className="w-full p-3 rounded-xl border border-gray-200 dark:border-gray-700 dark:bg-gray-700 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-300"
              required
            />
            <input
              type="number"
              name="amount"
              placeholder="Belopp (t.ex. 500)"
              value={newExpense.amount}
              onChange={handleInputChange}
              className="w-full p-3 rounded-xl border border-gray-200 dark:border-gray-700 dark:bg-gray-700 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-300"
              required
            />
            <div className="relative">
              <select
                name="payerId"
                value={newExpense.payerId}
                onChange={handleInputChange}
                className="w-full p-3 rounded-xl border border-gray-200 dark:border-gray-700 dark:bg-gray-700 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-300 appearance-none"
                required
              >
                <option value="" disabled>Vem betalade?</option>
                <option value={userId}>{userName}</option>
                <option value={partnerId}>{partnerName}</option>
              </select>
              <div className="absolute inset-y-0 right-0 flex items-center px-2 pointer-events-none">
                <svg className="h-5 w-5 text-gray-400 dark:text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M19 9l-7 7-7-7"></path>
                </svg>
              </div>
            </div>

            <div className="relative">
              <select
                name="splitType"
                value={newExpense.splitType}
                onChange={handleInputChange}
                className="w-full p-3 rounded-xl border border-gray-200 dark:border-gray-700 dark:bg-gray-700 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-300 appearance-none"
                required
              >
                <option value="even">Dela 50/50</option>
                <option value="salary">Dela baserat på nettolön</option>
              </select>
              <div className="absolute inset-y-0 right-0 flex items-center px-2 pointer-events-none">
                <svg className="h-5 w-5 text-gray-400 dark:text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M19 9l-7 7-7-7"></path>
                </svg>
              </div>
            </div>

            <button
              type="submit"
              className="w-full py-3 px-4 bg-gradient-to-r from-teal-400 to-emerald-500 text-white font-semibold rounded-xl shadow-lg hover:from-teal-500 hover:to-emerald-600 transition-transform transform hover:scale-105"
            >
              Lägg till kostnad
            </button>
          </form>
        </div>

        {balanceSummary && (
          <div className="p-4 sm:p-6 bg-white dark:bg-gray-800 rounded-3xl shadow-xl mb-6">
            <h2 className="text-xl sm:text-2xl font-semibold mb-4 text-gray-800 dark:text-white">Sammanställning</h2>
            <div className="border-t border-gray-200 dark:border-gray-700 pt-4">
              {Object.entries(balanceSummary.balances).map(([id, balance]) => (
                <div key={id} className="flex justify-between items-center py-2">
                  <span className="font-semibold text-lg">{id === userId ? userName : partnerName}</span>
                  <span className={`font-bold text-xl ${balance < 0 ? 'text-red-500' : 'text-emerald-500'}`}>
                    {balance > 0 ? '+' : ''}{balance.toFixed(2)} kr
                  </span>
                </div>
              ))}
            </div>

            <div className="border-t border-gray-200 dark:border-gray-700 pt-4 mt-4">
              <h3 className="text-lg sm:text-xl font-semibold mb-2 text-gray-800 dark:text-white">Vem ska swisha vem?</h3>
              {Object.keys(balanceSummary.whoOwesWhom).length === 0 ? (
                <p className="text-gray-500">Allt är jämnt!</p>
              ) : (
                <ul className="space-y-2">
                  {Object.entries(balanceSummary.whoOwesWhom).map(([receiverId, transfers]) => (
                    Object.entries(transfers).map(([payerId, amount]) => (
                      <li key={`${payerId}-${receiverId}`} className="p-3 bg-gray-100 dark:bg-gray-700 rounded-xl shadow-sm flex items-center justify-between text-sm">
                        <span>
                          {getPayerName(payerId)} ska swisha {getPayerName(receiverId)}
                        </span>
                        <span className="font-bold text-emerald-500">
                          {amount.toFixed(2)} kr
                        </span>
                      </li>
                    ))
                  ))}
                </ul>
              )}
            </div>

            <div className="mt-4">
              <button
                onClick={() => setShowExplanation(!showExplanation)}
                className="w-full text-center text-sm text-gray-500 dark:text-gray-400 hover:text-teal-500 transition-colors"
              >
                {showExplanation ? 'Dölj förklaring' : 'Hur fungerar detta?'}
              </button>
              {showExplanation && (
                <div className="mt-2 text-sm text-gray-500 dark:text-gray-400 p-3 bg-gray-100 dark:bg-gray-700 rounded-lg">
                  <p className="mb-2">
                    Appen beräknar den totala kostnaden och delar den på antalet användare för att få en genomsnittlig summa. Sedan jämförs varje persons totala utgifter med genomsnittet. Det resulterande beloppet visar hur mycket mer eller mindre varje person har betalat än genomsnittet. Om en person har betalat mer än genomsnittet, har de en positiv balans, och tvärtom. Detta ger en rättvis och korrekt beräkning.
                  </p>
                </div>
              )}
            </div>

          </div>
        )}

        <div className="p-4 sm:p-6 bg-white dark:bg-gray-800 rounded-3xl shadow-xl">
          <h2 className="text-xl sm:text-2xl font-semibold mb-4 text-gray-800 dark:text-white">Alla kostnader</h2>
          {expenses.length === 0 ? (
            <p className="text-gray-500">Inga kostnader har lagts till än.</p>
          ) : (
            <ul className="space-y-3">
              {expenses.map((expense) => (
                <li
                  key={expense.id}
                  className="p-4 bg-gray-100 dark:bg-gray-700 rounded-xl shadow-sm flex flex-col sm:flex-row justify-between items-start sm:items-center"
                >
                  <div className="flex-1 min-w-0">
                    <p className="font-medium text-lg truncate">{expense.description}</p>
                    <p className="text-sm text-gray-500 dark:text-gray-400 mt-1">
                      Betalades av: <span className="font-semibold">{getPayerName(expense.payerId)}</span>
                    </p>
                  </div>
                  <div className="mt-2 sm:mt-0 sm:ml-4 text-right">
                    <span className="font-bold text-xl">{expense.amount.toFixed(2)} kr</span>
                  </div>
                </li>
              ))}
            </ul>
          )}
        </div>
      </div>
    </div>
  );
};

export default App;

                        
