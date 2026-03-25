# java libraries

----

# Java Swing 

* Java Swing is the classic library for building Graphical User Interfaces (GUI) for desktop applications. 
* Released as part of the Java Foundation Classes (JFC), it was the industry standard for over a decade 
* before the shift toward web-based architectures.

1. Purpose and Main Aspects
--
* Swing’s primary goal is to allow developers to create windows, buttons, tables, and menus t
* hat look and behave consistently across all operating systems ("Write Once, Run Anywhere").

	* Lightweight Components: Unlike its predecessor (AWT), Swing doesn't rely on the host OS's native UI components. 
		* It "paints" its own buttons using Java 2D, offering more flexibility.
	* Pluggable Look and Feel: 
		* You can change the application's appearance (make it look like Windows, Mac, or a custom theme) without changing the underlying code logic.
	* Event-Driven: 
		* It uses the Observer Pattern. 
			* When a user interacts (e.g., clicks a button), an ActionListener is triggered to execute the response.
	* Single-Threaded Model: 
		* All UI updates must happen on a specific thread called the EDT (Event Dispatch Thread) to prevent race conditions.

2. Relevance Nowadays
--
* Swing is not the first choice for most new modern projects, but it remains highly relevant for three reasons:

	* Legacy Systems: 
		* Massive internal systems in banking, logistics, and government were built with Swing and require ongoing maintenance.
	* Developer Tools: 
		* Famous IDEs like IntelliJ IDEA and NetBeans are built using Swing.
	* Local Performance: 
		* It is still used for desktop tools that require direct access to local hardware or files where web latency is unacceptable.

* Note: While JavaFX is technically the modern successor to Swing, 
* both have largely lost ground to Web technologies (React, Vue) and Electron.

3. Relation/Comparison with Spring Boot
--
* they operate in completely different layers of an architecture.

| Feature | Java Swing | Spring Boot |
|---|---|---|
| Layer | Frontend (Desktop Client). | Backend (Server/API). |
| Execution | Runs on the user's local machine. | Runs on a server or in the cloud. |
| Interface | Windows, buttons, visual components. | JSON, XML, HTTP endpoints. |
| Communication | Direct memory/Local. | Network-based (REST/gRPC). |

Can they work together?
--
* Yes. 
	* A common modernization pattern involves a legacy Swing desktop app that acts as a client, 
	* making HTTP requests (using WebClient) to a modern Spring Boot backend.
* you can bootstrap a SpringApplicationContext inside a Swing application to use Dependency Injection and JPA, 
	* but Spring Boot's primary optimizations are for headless web services, not desktop window management
Aqui tens um exemplo prático de uma janela simples em Swing utilizando Lambdas (Java 8+) para tornar o código mais limpo.
Exemplo: Janela de Clique
Neste código, criamos um botão que, ao ser clicado, utiliza o JOptionPane para mostrar uma mensagem.

import javax.swing.*;import java.awt.*;
public class SwingLambdaExample {
    public static void main(String[] args) {
        // Swing não é thread-safe, por isso devemos iniciar na Event Dispatch Thread (EDT)
        SwingUtilities.invokeLater(() -> {
            
            // 1. Criar a Janela (JFrame)
            JFrame frame = new JFrame("Exemplo TDD/Swing");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 200);
            frame.setLayout(new FlowLayout());

            // 2. Criar o Botão
            JButton button = new JButton("Clica aqui!");

            // 3. O "Pulo do Gato": Usar Lambda para o evento de clique
            // Em vez de criar uma classe anónima pesada, usamos (e) -> { ... }
            button.addActionListener(e -> {
                JOptionPane.showMessageDialog(frame, "Botão clicado com sucesso!");
            });

            // 4. Adicionar à janela e mostrar
            frame.add(button);
            frame.setLocationRelativeTo(null); // Centrar no ecrã
            frame.setVisible(true);
        });
    }
}

Por que isto é importante para a tua entrevista?

   1. Thread Safety (SwingUtilities.invokeLater): Mostrar que sabes que o Swing é Single-Threaded. Tentar criar a interface diretamente na main pode causar erros visuais aleatórios. [1, 2]
   2. Modernização (Lambdas): Antigamente usava-se new ActionListener() { ... }. Usar Lambdas mostra que estás atualizado com o Java Moderno. [3, 4]
   3. Encapsulamento: Em projetos reais, a lógica do clique chamaria um Service (que poderia estar injetado via Spring, se integrares os dois). [5]

Comparação com o Mundo Real (Web)

* No Swing, o ActionListener é o equivalente ao onClick do React/JavaScript.
* O JFrame é o equivalente à window do browser.

Gostarias de ver como integrar o Spring Boot para injetar um @Service dentro desta classe Swing?



