# Constitución del proyecto

1. **Stack simple:** usar un solo lenguaje y añadir dependencias únicamente cuando una necesidad documentada no pueda resolverse con el stack existente.
2. **Spec y código:** toda funcionalidad debe estar especificada antes de implementarse y tener criterios de aceptación verificables.
3. **Lógica e interfaz:** la lógica de recordatorios debe poder probarse sin Telegram; la interfaz solo recibe entradas y presenta respuestas.
4. **Tests:** cada funcionalidad nueva y cada corrección de errores debe incluir pruebas automatizadas; las pruebas deben pasar antes de integrar cambios.
5. **Persistencia:** los recordatorios deben sobrevivir a reinicios y conservar su configuración; no se permite depender solo de memoria temporal.
6. **Idioma:** nombres de código y documentación técnica en inglés; mensajes del bot y comunicación con el usuario en español.
