<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rayito de Sol Continental</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f2f5;
        }
        .main-container {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 1rem;
        }
        .card {
            background-color: #fff;
            border-radius: 1.5rem;
            padding: 2.5rem;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            width: 100%;
            max-width: 28rem;
            text-align: center;
            animation: fadeIn 0.5s ease-out;
        }
        .logo {
            color: #ef4444;
            font-weight: 700;
        }
        .button {
            transition: all 0.3s ease;
        }
        .button:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .umbrella-card {
            background-color: #f9fafb;
            border-radius: 1rem;
            padding: 1rem;
            margin: 0.5rem 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-radius: 50%;
            border-top: 4px solid #ef4444;
            width: 24px;
            height: 24px;
            -webkit-animation: spin 2s linear infinite;
            animation: spin 2s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body class="bg-gray-100 font-sans">
    <div id="app" class="main-container bg-red-600 bg-opacity-90">
        <!-- Contenido de la aplicación se renderizará aquí por JavaScript -->
    </div>

    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot, collection, query, where, updateDoc, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Variables globales proporcionadas por el entorno de Canvas
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Inicializar Firebase
        let app, db, auth, userId;
        let isAuthReady = false;
        
        // Modal UI
        const createModal = (title, message, isError = false) => {
            const existingModal = document.getElementById('app-modal');
            if (existingModal) existingModal.remove();

            const modal = document.createElement('div');
            modal.id = 'app-modal';
            modal.className = 'fixed inset-0 bg-gray-600 bg-opacity-75 flex items-center justify-center p-4 z-50';
            modal.innerHTML = `
                <div class="bg-white rounded-xl p-6 max-w-sm w-full text-center shadow-lg">
                    <div class="flex flex-col items-center">
                        <div class="text-3xl mb-4 ${isError ? 'text-red-500' : 'text-green-500'}">
                            ${isError ? '⚠️' : '✅'}
                        </div>
                        <h3 class="text-lg font-bold mb-2 text-gray-900">${title}</h3>
                        <p class="text-sm text-gray-500 mb-4">${message}</p>
                        <button id="close-modal" class="w-full bg-red-500 text-white rounded-full py-2 font-semibold">Cerrar</button>
                    </div>
                </div>
            `;
            document.body.appendChild(modal);

            document.getElementById('close-modal').onclick = () => {
                modal.remove();
            };
        };

        const render = (html) => {
            const appDiv = document.getElementById('app');
            appDiv.innerHTML = html;
            // Re-attach event listeners after rendering
            attachEventListeners();
        };

        const renderLandingPage = () => {
            render(`
                <div class="card bg-yellow-300">
                    <div class="flex flex-col items-center mb-6">
                        <img src="https://placehold.co/100x100/FFF/000?text=☀️" alt="Logo de Rayito de Sol" class="rounded-full mb-4">
                        <h1 class="text-4xl font-extrabold text-red-600 logo mb-2">Rayito de Sol Continental</h1>
                        <p class="text-sm text-gray-700 font-semibold">
                            <span class="bg-yellow-100 text-yellow-800 px-2 py-1 rounded-full">¡Tu solución para la lluvia!</span>
                        </p>
                    </div>
                    <div class="space-y-4">
                        <p class="text-sm text-gray-600">
                            Servicio de alquiler de paraguas. Solo S/ 0.50 por un máximo de 4 días.
                            <br>Horario de atención: 7:00 a.m. - 10:00 p.m.
                        </p>
                        <button id="start-button" class="w-full bg-red-600 text-white font-bold py-3 px-6 rounded-full text-lg button">
                            Comenzar
                        </button>
                    </div>
                </div>
            `);
        };

        const renderLogin = () => {
            render(`
                <div class="card">
                    <h2 class="text-3xl font-bold mb-6 text-gray-900">Ingresa tu DNI</h2>
                    <p class="text-sm text-gray-600 mb-6">Para usar el servicio, necesitamos tu DNI (solo para identificación).</p>
                    <input id="dni-input" type="text" placeholder="DNI (8 dígitos)" class="w-full p-3 mb-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-red-500">
                    <button id="login-button" class="w-full bg-red-600 text-white font-bold py-3 px-6 rounded-full text-lg button">
                        Continuar
                    </button>
                    <button id="back-button" class="w-full mt-4 text-gray-500 font-semibold text-sm">
                        Volver
                    </button>
                </div>
            `);
        };
        
        const renderMainApp = async () => {
            render(`
                <div class="card">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-2xl font-bold text-gray-900">Bienvenido/a</h2>
                        <button id="logout-button" class="text-sm text-gray-500 hover:text-red-500">
                            Cerrar Sesión
                        </button>
                    </div>
                    <div class="text-xs text-gray-400 mb-4 text-left">
                        <p>ID de Usuario: <span id="user-id-display">${userId || 'Cargando...'}</span></p>
                    </div>
                    <div id="content-container">
                        <div class="flex items-center justify-center p-8">
                            <div class="loader"></div>
                        </div>
                    </div>
                </div>
            `);
            const userDoc = await getDoc(doc(db, 'artifacts', appId, 'users', userId));
            if (!userDoc.exists()) {
                await setDoc(doc(db, 'artifacts', appId, 'users', userId), {
                    dni: document.getElementById('dni-input')?.value || 'N/A',
                    hasUmbrella: false,
                    lastRent: null
                });
            }

            // Listen for user data changes
            onSnapshot(doc(db, 'artifacts', appId, 'users', userId), (docSnapshot) => {
                if (docSnapshot.exists()) {
                    const userData = docSnapshot.data();
                    const contentContainer = document.getElementById('content-container');
                    if (userData.hasUmbrella) {
                        renderReturnView(userData, contentContainer);
                    } else {
                        renderUmbrellaListView(contentContainer);
                    }
                }
            });
        };

        const renderUmbrellaListView = (container) => {
            container.innerHTML = `
                <h3 class="text-xl font-semibold mb-4 text-gray-800">Paraguas Disponibles</h3>
                <p class="text-sm text-gray-500 mb-4">
                    Selecciona un paraguas para alquilar (S/ 0.50).
                </p>
                <div id="umbrella-list" class="space-y-2 max-h-60 overflow-y-auto pr-2">
                    <div class="flex items-center justify-center p-8">
                        <div class="loader"></div>
                    </div>
                </div>
                <div id="operational-hours" class="mt-4 p-3 rounded-lg text-sm bg-blue-100 text-blue-700">
                    Horario: 7:00 a.m. - 10:00 p.m.
                </div>
            `;
            // Se ha cambiado la ruta de la colección de paraguas para solucionar el error de permisos.
            // Ahora la lista de paraguas es privada para cada usuario.
            onSnapshot(collection(db, 'artifacts', appId, 'users', userId, 'umbrellas'), (snapshot) => {
                const umbrellaListDiv = document.getElementById('umbrella-list');
                const umbrellas = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                const availableUmbrellas = umbrellas.filter(u => u.isAvailable);

                if (availableUmbrellas.length > 0) {
                    umbrellaListDiv.innerHTML = availableUmbrellas.map(umbrella => `
                        <div class="umbrella-card bg-gray-50 border border-gray-200">
                            <span class="flex items-center">
                                <span class="w-4 h-4 rounded-full mr-2" style="background-color: ${umbrella.color.toLowerCase()};"></span>
                                <span class="font-medium text-gray-700">${umbrella.color}</span>
                            </span>
                            <button class="rent-umbrella-button bg-green-500 text-white font-semibold text-sm py-2 px-4 rounded-full button" data-id="${umbrella.id}">
                                Alquilar
                            </button>
                        </div>
                    `).join('');
                } else {
                    umbrellaListDiv.innerHTML = `
                        <p class="text-center text-gray-500">No hay paraguas disponibles en este momento.</p>
                    `;
                }
            });
        };

        const renderReturnView = (userData, container) => {
            const rentalDate = new Date(userData.lastRent.seconds * 1000);
            const now = new Date();
            const maxDuration = 4 * 24 * 60 * 60 * 1000; // 4 días en milisegundos
            const timeElapsed = now.getTime() - rentalDate.getTime();
            const timeRemaining = Math.max(0, maxDuration - timeElapsed);

            const daysRemaining = Math.floor(timeRemaining / (1000 * 60 * 60 * 24));
            const hoursRemaining = Math.floor((timeRemaining % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
            const minutesRemaining = Math.floor((timeRemaining % (1000 * 60 * 60)) / (1000 * 60));

            container.innerHTML = `
                <h3 class="text-xl font-semibold mb-4 text-gray-800">Tienes un paraguas</h3>
                <p class="text-sm text-gray-500 mb-4">
                    ¡Gracias por usar nuestro servicio! Por favor, devuélvelo a tiempo.
                </p>
                <div class="p-4 bg-gray-100 rounded-lg mb-4">
                    <p class="font-bold text-gray-700">Tiempo restante:</p>
                    <p class="text-lg text-green-600">${daysRemaining} días, ${hoursRemaining} horas y ${minutesRemaining} minutos</p>
                </div>
                <div class="p-4 bg-red-100 text-red-700 rounded-lg mb-4">
                    <label class="flex items-center justify-center">
                        <input type="checkbox" id="damaged-checkbox" class="mr-2">
                        El paraguas está dañado (S/ 5.00)
                    </label>
                </div>
                <button id="return-button" class="w-full bg-red-600 text-white font-bold py-3 px-6 rounded-full text-lg button">
                    Completar Devolución
                </button>
            `;
        };

        const checkOperationalHours = () => {
            const now = new Date();
            const hour = now.getHours();
            return hour >= 7 && hour < 22; // 7 a.m. a 10 p.m.
        };

        const attachEventListeners = () => {
            document.getElementById('start-button')?.addEventListener('click', renderLogin);
            document.getElementById('back-button')?.addEventListener('click', renderLandingPage);

            document.getElementById('login-button')?.addEventListener('click', async () => {
                const dniInput = document.getElementById('dni-input');
                const dni = dniInput.value.trim();

                if (dni.length !== 8 || !/^\d+$/.test(dni)) {
                    createModal('Error de DNI', 'Por favor, ingresa un DNI válido de 8 dígitos.', true);
                    return;
                }
                
                try {
                    await signInWithCustomToken(auth, initialAuthToken);
                    renderMainApp();
                } catch (error) {
                    console.error("Error signing in with custom token:", error);
                    createModal('Error de Autenticación', 'No se pudo autenticar. Por favor, inténtalo de nuevo.', true);
                }
            });

            document.getElementById('logout-button')?.addEventListener('click', async () => {
                await signOut(auth);
                renderLandingPage();
            });

            document.querySelectorAll('.rent-umbrella-button').forEach(button => {
                button.addEventListener('click', async (e) => {
                    const umbrellaId = e.target.getAttribute('data-id');
                    const now = new Date();
                    const currentHour = now.getHours();

                    if (!checkOperationalHours()) {
                        createModal('Fuera de Horario', 'El servicio de alquiler de paraguas solo está disponible de 7 a.m. a 10 p.m.', true);
                        return;
                    }

                    // Check if user already has an umbrella
                    const userDocRef = doc(db, 'artifacts', appId, 'users', userId);
                    const userDoc = await getDoc(userDocRef);
                    const userData = userDoc.data();
                    if (userData && userData.hasUmbrella) {
                        createModal('Error', 'Solo puedes prestar un paraguas por persona.', true);
                        return;
                    }

                    // Check if umbrella is still available
                    // Se ha actualizado la ruta para acceder a la colección de paraguas del usuario
                    const umbrellaDocRef = doc(db, 'artifacts', appId, 'users', userId, 'umbrellas', umbrellaId);
                    const umbrellaDoc = await getDoc(umbrellaDocRef);
                    if (!umbrellaDoc.exists() || !umbrellaDoc.data().isAvailable) {
                        createModal('No Disponible', 'Este paraguas ya no está disponible. Por favor, elige otro.', true);
                        return;
                    }

                    // Mock Yape payment
                    createModal('Pago con Yape', 'Simulando el pago de S/ 0.50 con Yape...', false);

                    setTimeout(async () => {
                        try {
                            // Update user data
                            await updateDoc(userDocRef, {
                                hasUmbrella: true,
                                lastRent: new Date(),
                                umbrellaId: umbrellaId
                            });

                            // Update umbrella data
                            await updateDoc(umbrellaDocRef, {
                                isAvailable: false,
                                lastRentedBy: userId
                            });

                            createModal('¡Éxito!', 'Paraguas alquilado exitosamente. ¡Que te diviertas!', false);
                        } catch (error) {
                            console.error("Error during rental:", error);
                            createModal('Error', 'Hubo un problema con el alquiler. Intenta de nuevo.', true);
                        }
                    }, 2000);
                });
            });

            document.getElementById('return-button')?.addEventListener('click', async () => {
                const damagedCheckbox = document.getElementById('damaged-checkbox');
                const isDamaged = damagedCheckbox.checked;

                const userDocRef = doc(db, 'artifacts', appId, 'users', userId);
                const userDoc = await getDoc(userDocRef);
                const userData = userDoc.data();
                const umbrellaId = userData.umbrellaId;

                const cost = isDamaged ? 5.00 : 0.00;
                const message = isDamaged
                    ? `Simulando el pago de S/ ${cost.toFixed(2)} por daños.`
                    : 'Simulando devolución del paraguas...';

                createModal('Devolución', message, false);

                setTimeout(async () => {
                    try {
                        // Se ha actualizado la ruta para acceder a la colección de paraguas del usuario
                        const umbrellaDocRef = doc(db, 'artifacts', appId, 'users', userId, 'umbrellas', umbrellaId);

                        // Update user data
                        await updateDoc(userDocRef, {
                            hasUmbrella: false,
                            umbrellaId: null
                        });

                        // Update umbrella data
                        await updateDoc(umbrellaDocRef, {
                            isAvailable: true,
                            isDamaged: isDamaged
                        });

                        createModal('Devolución Exitosa', '¡El paraguas ha sido devuelto! ¡Gracias por usar Rayito de Sol!', false);
                    } catch (error) {
                        console.error("Error during return:", error);
                        createModal('Error', 'Hubo un problema con la devolución. Intenta de nuevo.', true);
                    }
                }, 2000);
            });
        };

        // Initialize App
        document.addEventListener('DOMContentLoaded', async () => {
            try {
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        isAuthReady = true;
                        // Se ha cambiado la ruta de la colección de paraguas para que sea privada.
                        const umbrellasCollectionRef = collection(db, 'artifacts', appId, 'users', userId, 'umbrellas');
                        const snapshot = await getDocs(umbrellasCollectionRef);
                        if (snapshot.empty) {
                            const umbrellaColors = ['Amarillo', 'Azul', 'Rojo', 'Verde', 'Negro', 'Blanco', 'Morado', 'Naranja'];
                            for (let i = 0; i < umbrellaColors.length; i++) {
                                await setDoc(doc(umbrellasCollectionRef, `umbrella-${i + 1}`), {
                                    color: umbrellaColors[i],
                                    isAvailable: true,
                                    isDamaged: false
                                });
                            }
                        }
                        renderMainApp();
                    } else {
                        renderLandingPage();
                    }
                });

            } catch (error) {
                console.error("Firebase initialization failed:", error);
                render(`
                    <div class="card bg-red-100 border-l-4 border-red-500 text-red-700 p-4" role="alert">
                        <p class="font-bold">Error de Carga</p>
                        <p>No se pudo conectar a la base de datos. Por favor, inténtalo de nuevo más tarde.</p>
                    </div>
                `);
            }
        });
    </script>
</body>
</html>
