<?php

// --- 1. Class: TicketType ---
// Represents different types of tickets available (e.g., Adult, Child)
class TicketType {
    public $id;
    public $name;
    public $price;

    public function __construct($id, $name, $price) {
        $this->id = $id;
        $this->name = $name;
        $this->price = $price;
    }

    public function display() {
        return "ID: {$this->id}, Tipo: {$this->name}, Precio: $" . number_format($this->price, 2);
    }
}

// --- 2. Class: User ---
// Represents a user who can make reservations
class User {
    public $id;
    public $name;
    public $email;
    public $phone;

    public function __construct($id, $name, $email, $phone) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
        $this->phone = $phone;
    }

    public function display() {
        return "ID: {$this->id}, Nombre: {$this->name}, Email: {$this->email}, Teléfono: {$this->phone}";
    }
}

// --- 3. Class: Date (incorporates Capacity) ---
// Represents a specific date with its capacity and list of reservations
class Date {
    public $dateString; // e.g., '2025-12-25'
    public $totalCapacity;
    public $availableCapacity;
    public $reservations = []; // Array to store Reservation objects

    public function __construct($dateString, $totalCapacity) {
        if (!DateTime::createFromFormat('Y-m-d', $dateString)) {
            throw new Exception("Formato de fecha inválido. Use YYYY-MM-DD.");
        }
        $this->dateString = $dateString;
        $this->totalCapacity = $totalCapacity;
        $this->availableCapacity = $totalCapacity;
    }

    public function addReservation(Reservation $reservation) {
        // Here you would add logic to check if capacity allows
        // For simplicity, we'll assume it's checked before calling this method
        $this->reservations[] = $reservation;
        $this->availableCapacity -= $reservation->getTotalTickets();
        return true;
    }

    public function removeReservation(Reservation $reservationToRemove) {
        foreach ($this->reservations as $key => $reservation) {
            if ($reservation->id === $reservationToRemove->id) {
                unset($this->reservations[$key]);
                $this->availableCapacity += $reservation->getTotalTickets();
                // Re-index array to avoid gaps
                $this->reservations = array_values($this->reservations);
                return true;
            }
        }
        return false;
    }

    public function display() {
        return "Fecha: {$this->dateString}, Capacidad Total: {$this->totalCapacity}, Disponible: {$this->availableCapacity}";
    }
}

// --- 4. Class: Reservation ---
// Links a user, a date, and specific ticket types/quantities
class Reservation {
    public $id;
    public $user; // User object
    public $date; // Date object
    public $tickets = []; // Associative array: ['ticketTypeId' => quantity]
    public $totalPrice;

    private static $nextReservationId = 1;

    public function __construct(User $user, Date $date) {
        $this->id = self::$nextReservationId++;
        $this->user = $user;
        $this->date = $date;
        $this->totalPrice = 0;
    }

    public function addTicket(TicketType $ticketType, $quantity) {
        if ($quantity <= 0) {
            echo "La cantidad de tickets debe ser mayor que cero.\n";
            return false;
        }
        $this->tickets[$ticketType->id] = ($this->tickets[$ticketType->id] ?? 0) + $quantity;
        $this->totalPrice += ($ticketType->price * $quantity);
        return true;
    }

    public function getTotalTickets() {
        return array_sum($this->tickets);
    }

    public function display($ticketTypesMap) { // $ticketTypesMap is an array of TicketType objects keyed by ID
        $ticketDetails = [];
        foreach ($this->tickets as $ticketTypeId => $quantity) {
            if (isset($ticketTypesMap[$ticketTypeId])) {
                $ticketName = $ticketTypesMap[$ticketTypeId]->name;
                $ticketDetails[] = "{$quantity} x {$ticketName}";
            }
        }
        $ticketsString = implode(", ", $ticketDetails);
        return "ID Reserva: {$this->id}, Usuario: {$this->user->name}, Fecha: {$this->date->dateString}, Tickets: [{$ticketsString}], Total: $" . number_format($this->totalPrice, 2);
    }
}

// --- 5. Class: Park ---
// Manages all other entities and provides methods for system operations
class Park {
    public $name;
    public $users = []; // Array of User objects
    public $ticketTypes = []; // Array of TicketType objects
    public $dates = []; // Array of Date objects
    public $reservations = []; // Array of Reservation objects

    public function __construct($name) {
        $this->name = $name;
    }

    // --- User Management ---
    public function addUser(User $user) {
        $this->users[$user->id] = $user;
        echo "Usuario '{$user->name}' agregado.\n";
    }

    public function updateUser($userId, $newName, $newEmail, $newPhone) {
        if (isset($this->users[$userId])) {
            $this->users[$userId]->name = $newName;
            $this->users[$userId]->email = $newEmail;
            $this->users[$userId]->phone = $newPhone;
            echo "Usuario ID {$userId} actualizado.\n";
            return true;
        }
        echo "Usuario ID {$userId} no encontrado.\n";
        return false;
    }

    public function deleteUser($userId) {
        if (isset($this->users[$userId])) {
            // Before deleting user, check for associated reservations
            foreach ($this->reservations as $reservationId => $reservation) {
                if ($reservation->user->id === $userId) {
                    echo "No se puede eliminar el usuario ID {$userId}. Tiene reservas asociadas.\n";
                    return false;
                }
            }
            unset($this->users[$userId]);
            echo "Usuario ID {$userId} eliminado.\n";
            return true;
        }
        echo "Usuario ID {$userId} no encontrado.\n";
        return false;
    }

    public function getUser($userId) {
        return $this->users[$userId] ?? null;
    }

    public function listUsers() {
        if (empty($this->users)) {
            echo "No hay usuarios registrados.\n";
            return;
        }
        echo "\n--- Listado de Usuarios ---\n";
        foreach ($this->users as $user) {
            echo $user->display() . "\n";
        }
    }

    // --- Ticket Type Management ---
    public function addTicketType(TicketType $ticketType) {
        $this->ticketTypes[$ticketType->id] = $ticketType;
        echo "Tipo de entrada '{$ticketType->name}' agregado.\n";
    }

    public function updateTicketType($typeId, $newName, $newPrice) {
        if (isset($this->ticketTypes[$typeId])) {
            $this->ticketTypes[$typeId]->name = $newName;
            $this->ticketTypes[$typeId]->price = $newPrice;
            echo "Tipo de entrada ID {$typeId} actualizado.\n";
            return true;
        }
        echo "Tipo de entrada ID {$typeId} no encontrado.\n";
        return false;
    }

    public function deleteTicketType($typeId) {
        if (isset($this->ticketTypes[$typeId])) {
            // Check for associated reservations before deleting
            foreach ($this->reservations as $reservation) {
                if (isset($reservation->tickets[$typeId])) {
                    echo "No se puede eliminar el tipo de entrada ID {$typeId}. Está asociado a reservas.\n";
                    return false;
                }
            }
            unset($this->ticketTypes[$typeId]);
            echo "Tipo de entrada ID {$typeId} eliminado.\n";
            return true;
        }
        echo "Tipo de entrada ID {$typeId} no encontrado.\n";
        return false;
    }

    public function getTicketType($typeId) {
        return $this->ticketTypes[$typeId] ?? null;
    }

    public function listTicketTypes() {
        if (empty($this->ticketTypes)) {
            echo "No hay tipos de entradas registrados.\n";
            return;
        }
        echo "\n--- Listado de Tipos de Entradas ---\n";
        foreach ($this->ticketTypes as $type) {
            echo $type->display() . "\n";
        }
    }

    // --- Date Management (Capacity included) ---
    public function addDate(Date $date) {
        if (isset($this->dates[$date->dateString])) {
            echo "La fecha {$date->dateString} ya existe.\n";
            return false;
        }
        $this->dates[$date->dateString] = $date;
        echo "Fecha {$date->dateString} con capacidad {$date->totalCapacity} agregada.\n";
        return true;
    }

    public function updateDateCapacity($dateString, $newCapacity) {
        if (isset($this->dates[$dateString])) {
            if ($newCapacity < $this->dates[$dateString]->totalCapacity - $this->dates[$dateString]->availableCapacity) {
                echo "La nueva capacidad ({$newCapacity}) es menor que las reservas existentes ({$this->dates[$dateString]->totalCapacity - $this->dates[$dateString]->availableCapacity}). No se puede actualizar.\n";
                return false;
            }
            $this->dates[$dateString]->totalCapacity = $newCapacity;
            $this->dates[$dateString]->availableCapacity = $newCapacity - (count($this->dates[$dateString]->reservations)); // Recalculate available capacity
            echo "Capacidad de la fecha {$dateString} actualizada a {$newCapacity}.\n";
            return true;
        }
        echo "Fecha {$dateString} no encontrada.\n";
        return false;
    }

    public function deleteDate($dateString) {
        if (isset($this->dates[$dateString])) {
            if (!empty($this->dates[$dateString]->reservations)) {
                echo "No se puede eliminar la fecha {$dateString}. Tiene reservas asociadas.\n";
                return false;
            }
            unset($this->dates[$dateString]);
            echo "Fecha {$dateString} eliminada.\n";
            return true;
        }
        echo "Fecha {$dateString} no encontrada.\n";
        return false;
    }

    public function getDate($dateString) {
        return $this->dates[$dateString] ?? null;
    }

    public function listDates() {
        if (empty($this->dates)) {
            echo "No hay fechas registradas.\n";
            return;
        }
        echo "\n--- Listado de Fechas y Capacidades ---\n";
        foreach ($this->dates as $date) {
            echo $date->display() . "\n";
        }
    }

    // --- Reservation Management ---
    public function createReservation($userId, $dateString, $ticketQuantities) {
        $user = $this->getUser($userId);
        $date = $this->getDate($dateString);

        if (!$user) {
            echo "Error: Usuario ID {$userId} no encontrado.\n";
            return null;
        }
        if (!$date) {
            echo "Error: Fecha {$dateString} no encontrada.\n";
            return null;
        }

        $totalRequestedTickets = array_sum($ticketQuantities);
        if ($date->availableCapacity < $totalRequestedTickets) {
            echo "Error: No hay suficiente capacidad disponible para la fecha {$dateString}. (Solicitados: {$totalRequestedTickets}, Disponibles: {$date->availableCapacity})\n";
            return null;
        }

        $reservation = new Reservation($user, $date);
        foreach ($ticketQuantities as $ticketTypeId => $quantity) {
            $ticketType = $this->getTicketType($ticketTypeId);
            if (!$ticketType) {
                echo "Advertencia: Tipo de entrada ID {$ticketTypeId} no encontrado. No se agregará a la reserva.\n";
                continue;
            }
            $reservation->addTicket($ticketType, $quantity);
        }

        if ($date->addReservation($reservation)) {
            $this->reservations[$reservation->id] = $reservation;
            echo "Reserva ID {$reservation->id} creada exitosamente.\n";
            return $reservation;
        } else {
            echo "Error al crear la reserva para la fecha.\n";
            return null;
        }
    }

    public function updateReservation($reservationId, $newTicketQuantities = []) {
        $reservation = $this->getReservation($reservationId);
        if (!$reservation) {
            echo "Reserva ID {$reservationId} no encontrada.\n";
            return false;
        }

        // For simplicity, updating reservations involves re-calculating everything.
        // A more robust system would handle partial updates carefully.
        $oldTotalTickets = $reservation->getTotalTickets();
        $oldDate = $reservation->date;

        // Create a temporary new reservation to calculate new totals
        $tempReservation = new Reservation($reservation->user, $oldDate);
        foreach ($newTicketQuantities as $ticketTypeId => $quantity) {
            $ticketType = $this->getTicketType($ticketTypeId);
            if (!$ticketType) {
                echo "Advertencia: Tipo de entrada ID {$ticketTypeId} no encontrado. No se agregará a la actualización.\n";
                continue;
            }
            $tempReservation->addTicket($ticketType, $quantity);
        }
        $newTotalTickets = $tempReservation->getTotalTickets();

        // Check if new capacity allows the change
        $capacityChange = $newTotalTickets - $oldTotalTickets;
        if ($oldDate->availableCapacity < $capacityChange) {
            echo "No hay suficiente capacidad para actualizar la reserva. (Necesarios: {$capacityChange}, Disponibles: {$oldDate->availableCapacity})\n";
            return false;
        }

        // If capacity allows, proceed with update
        $oldDate->availableCapacity += $oldTotalTickets; // Return old tickets to capacity
        $reservation->tickets = []; // Clear old tickets
        $reservation->totalPrice = 0; // Reset price

        foreach ($newTicketQuantities as $ticketTypeId => $quantity) {
            $ticketType = $this->getTicketType($ticketTypeId);
            if ($ticketType) {
                $reservation->addTicket($ticketType, $quantity);
            }
        }
        $oldDate->availableCapacity -= $newTotalTickets; // Subtract new tickets from capacity

        echo "Reserva ID {$reservationId} actualizada exitosamente.\n";
        return true;
    }


    public function deleteReservation($reservationId) {
        if (isset($this->reservations[$reservationId])) {
            $reservation = $this->reservations[$reservationId];
            $reservation->date->removeReservation($reservation); // Update date's capacity
            unset($this->reservations[$reservationId]);
            echo "Reserva ID {$reservationId} eliminada.\n";
            return true;
        }
        echo "Reserva ID {$reservationId} no encontrada.\n";
        return false;
    }

    public function getReservation($reservationId) {
        return $this->reservations[$reservationId] ?? null;
    }

    public function listReservations() {
        if (empty($this->reservations)) {
            echo "No hay reservas registradas.\n";
            return;
        }
        echo "\n--- Listado de Reservas ---\n";
        // Create a map of ticket types for easier display
        $ticketTypesMap = [];
        foreach ($this->ticketTypes as $type) {
            $ticketTypesMap[$type->id] = $type;
        }
        foreach ($this->reservations as $reservation) {
            echo $reservation->display($ticketTypesMap) . "\n";
        }
    }
}

// --- Menu Functions ---

function displayMainMenu() {
    echo "\n--- Menú Principal del Parque Termal ---\n";
    echo "1. Administrar Usuarios\n";
    echo "2. Administrar Tipos de Entradas\n";
    echo "3. Administrar Fechas y Capacidades\n";
    echo "4. Administrar Reservas\n";
    echo "5. Salir\n";
    echo "Elija una opción: ";
}

function displaySubMenu($entityName) {
    echo "\n--- Administrar {$entityName} ---\n";
    echo "1. Ingresar {$entityName}\n";
    echo "2. Modificar {$entityName}\n";
    echo "3. Eliminar {$entityName}\n";
    echo "4. Listar {$entityName}s\n";
    echo "5. Volver al Menú Principal\n";
    echo "Elija una opción: ";
}

function getUserInput($prompt) {
    echo $prompt;
    return trim(fgets(STDIN));
}

// --- Main Program Logic ---

$park = new Park("Parque Termal Oasis");

// Initial data (for testing)
$park->addTicketType(new TicketType(1, "Adulto", 35.00));
$park->addTicketType(new TicketType(2, "Niño (4-12)", 20.00));
$park->addTicketType(new TicketType(3, "Tercera Edad", 25.00));

$park->addUser(new User(101, "Juan Perez", "juan@example.com", "1122334455"));
$park->addUser(new User(102, "Maria Garcia", "maria@example.com", "1166778899"));

try {
    $park->addDate(new Date('2025-07-01', 100));
    $park->addDate(new Date('2025-07-02', 80));
    $park->addDate(new Date('2025-07-03', 120));
} catch (Exception $e) {
    echo "Error al agregar fecha: " . $e->getMessage() . "\n";
}


while (true) {
    displayMainMenu();
    $mainChoice = getUserInput("");

    switch ($mainChoice) {
        case '1': // Administrar Usuarios
            while (true) {
                displaySubMenu("Usuario");
                $subChoice = getUserInput("");
                switch ($subChoice) {
                    case '1': // Ingresar Usuario
                        $id = (int) getUserInput("Ingrese ID de usuario: ");
                        $name = getUserInput("Ingrese nombre: ");
                        $email = getUserInput("Ingrese email: ");
                        $phone = getUserInput("Ingrese teléfono: ");
                        $park->addUser(new User($id, $name, $email, $phone));
                        break;
                    case '2': // Modificar Usuario
                        $id = (int) getUserInput("Ingrese ID de usuario a modificar: ");
                        $name = getUserInput("Ingrese nuevo nombre (dejar vacío para no cambiar): ");
                        $email = getUserInput("Ingrese nuevo email (dejar vacío para no cambiar): ");
                        $phone = getUserInput("Ingrese nuevo teléfono (dejar vacío para no cambiar): ");
                        $userToUpdate = $park->getUser($id);
                        if ($userToUpdate) {
                            $park->updateUser($id, $name ?: $userToUpdate->name, $email ?: $userToUpdate->email, $phone ?: $userToUpdate->phone);
                        } else {
                            echo "Usuario no encontrado.\n";
                        }
                        break;
                    case '3': // Eliminar Usuario
                        $id = (int) getUserInput("Ingrese ID de usuario a eliminar: ");
                        $park->deleteUser($id);
                        break;
                    case '4': // Listar Usuarios
                        $park->listUsers();
                        break;
                    case '5':
                        break 2; // Exit inner loop and go back to main menu
                    default:
                        echo "Opción inválida. Intente de nuevo.\n";
                }
            }
            break;

        case '2': // Administrar Tipos de Entradas
            while (true) {
                displaySubMenu("Tipo de Entrada");
                $subChoice = getUserInput("");
                switch ($subChoice) {
                    case '1': // Ingresar Tipo de Entrada
                        $id = (int) getUserInput("Ingrese ID de tipo de entrada: ");
                        $name = getUserInput("Ingrese nombre del tipo de entrada: ");
                        $price = (float) getUserInput("Ingrese precio: ");
                        $park->addTicketType(new TicketType($id, $name, $price));
                        break;
                    case '2': // Modificar Tipo de Entrada
                        $id = (int) getUserInput("Ingrese ID de tipo de entrada a modificar: ");
                        $name = getUserInput("Ingrese nuevo nombre (dejar vacío para no cambiar): ");
                        $price = getUserInput("Ingrese nuevo precio (dejar vacío para no cambiar): ");
                        $typeToUpdate = $park->getTicketType($id);
                        if ($typeToUpdate) {
                            $park->updateTicketType($id, $name ?: $typeToUpdate->name, $price === '' ? $typeToUpdate->price : (float)$price);
                        } else {
                            echo "Tipo de entrada no encontrado.\n";
                        }
                        break;
                    case '3': // Eliminar Tipo de Entrada
                        $id = (int) getUserInput("Ingrese ID de tipo de entrada a eliminar: ");
                        $park->deleteTicketType($id);
                        break;
                    case '4': // Listar Tipos de Entradas
                        $park->listTicketTypes();
                        break;
                    case '5':
                        break 2;
                    default:
                        echo "Opción inválida. Intente de nuevo.\n";
                }
            }
            break;

        case '3': // Administrar Fechas y Capacidades
            while (true) {
                displaySubMenu("Fecha y Capacidad");
                $subChoice = getUserInput("");
                switch ($subChoice) {
                    case '1': // Ingresar Fecha
                        $dateString = getUserInput("Ingrese fecha (YYYY-MM-DD): ");
                        $capacity = (int) getUserInput("Ingrese capacidad total: ");
                        try {
                            $park->addDate(new Date($dateString, $capacity));
                        } catch (Exception $e) {
                            echo "Error: " . $e->getMessage() . "\n";
                        }
                        break;
                    case '2': // Modificar Capacidad de Fecha
                        $dateString = getUserInput("Ingrese fecha a modificar (YYYY-MM-DD): ");
                        $newCapacity = (int) getUserInput("Ingrese nueva capacidad total: ");
                        $park->updateDateCapacity($dateString, $newCapacity);
                        break;
                    case '3': // Eliminar Fecha
                        $dateString = getUserInput("Ingrese fecha a eliminar (YYYY-MM-DD): ");
                        $park->deleteDate($dateString);
                        break;
                    case '4': // Listar Fechas y Capacidades
                        $park->listDates();
                        break;
                    case '5':
                        break 2;
                    default:
                        echo "Opción inválida. Intente de nuevo.\n";
                }
            }
            break;

        case '4': // Administrar Reservas
            while (true) {
                echo "\n--- Administrar Reservas ---\n";
                echo "1. Crear Reserva\n";
                echo "2. Modificar Reserva\n";
                echo "3. Eliminar Reserva\n";
                echo "4. Listar Reservas\n";
                echo "5. Volver al Menú Principal\n";
                echo "Elija una opción: ";
                $subChoice = getUserInput("");

                switch ($subChoice) {
                    case '1': // Crear Reserva
                        $park->listUsers();
                        $userId = (int) getUserInput("Ingrese ID de usuario para la reserva: ");
                        $park->listDates();
                        $dateString = getUserInput("Ingrese fecha para la reserva (YYYY-MM-DD): ");
                        $park->listTicketTypes();

                        $ticketQuantities = [];
                        while (true) {
                            $typeId = (int) getUserInput("Ingrese ID de tipo de entrada (0 para terminar): ");
                            if ($typeId === 0) break;
                            $quantity = (int) getUserInput("Ingrese cantidad para este tipo de entrada: ");
                            if ($quantity > 0) {
                                $ticketQuantities[$typeId] = $quantity;
                            } else {
                                echo "Cantidad debe ser mayor que 0.\n";
                            }
                        }
                        if (!empty($ticketQuantities)) {
                            $park->createReservation($userId, $dateString, $ticketQuantities);
                        } else {
                            echo "No se agregaron tickets a la reserva.\n";
                        }
                        break;
                    case '2': // Modificar Reserva
                        $park->listReservations();
                        $reservationId = (int) getUserInput("Ingrese ID de la reserva a modificar: ");
                        $existingReservation = $park->getReservation($reservationId);

                        if ($existingReservation) {
                            echo "Reserva actual: " . $existingReservation->display($park->ticketTypes) . "\n";
                            echo "Ingrese nuevas cantidades de tickets (dejar vacío para no cambiar, 0 para eliminar un tipo):\n";
                            $newTicketQuantities = [];
                            foreach ($park->ticketTypes as $type) {
                                $currentQuantity = $existingReservation->tickets[$type->id] ?? 0;
                                $input = getUserInput("Cantidad de {$type->name} (actual: {$currentQuantity}, 0 para quitar): ");
                                if ($input !== '') {
                                    $newQuantity = (int)$input;
                                    if ($newQuantity >= 0) {
                                        $newTicketQuantities[$type->id] = $newQuantity;
                                    }
                                } else {
                                    $newTicketQuantities[$type->id] = $currentQuantity; // Keep existing quantity if not changed
                                }
                            }
                             $park->updateReservation($reservationId, array_filter($newTicketQuantities, fn($q) => $q > 0)); // Filter out 0 quantities
                        } else {
                            echo "Reserva no encontrada.\n";
                        }
                        break;
                    case '3': // Eliminar Reserva
                        $park->listReservations();
                        $reservationId = (int) getUserInput("Ingrese ID de la reserva a eliminar: ");
                        $park->deleteReservation($reservationId);
                        break;
                    case '4': // Listar Reservas
                        $park->listReservations();
                        break;
                    case '5':
                        break 2;
                    default:
                        echo "Opción inválida. Intente de nuevo.\n";
                }
            }
            break;

        case '5': // Salir
            echo "Saliendo del programa. ¡Hasta luego!\n";
            exit();
        default:
            echo "Opción inválida. Por favor, elija una opción del 1 al 5.\n";
    }
}

?>
