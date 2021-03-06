1.	Melden Sie sich online beim [Microsoft Azure-Portal](https://portal.azure.com/) an.
2.	Klicken Sie in der Indexleiste auf **Neu**, klicken Sie auf **Daten und Speicher** und dann auf **Azure DocumentDB**.

	![Screenshot des Azure-Portals bei Erstellen einer Datenbank, markiert sind die Schaltfläche „Neu“ sowie „Daten und Speicher“ auf dem Blatt „Erstellen“ und „Azure DocumentDB“ auf dem Blatt „Daten und Speicher“](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png)

3. Geben Sie auf dem Blatt **Neues DocumentDB-Konto** die gewünschte Konfiguration für das DocumentDB-Konto an.

	![Screenshot des Blatts "Neue DocumentDB"](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-2.png)


	- Geben Sie im Feld **ID** einen Namen zur Identifizierung des DocumentDB-Kontos ein. Wenn die **ID** überprüft wurde, wird im Feld **ID** ein grünes Häkchen angezeigt. Der Wert für **ID** wird zum Hostnamen innerhalb des URI. Die **ID** darf nur Kleinbuchstaben, Ziffern und den Bindestrich "-" enthalten und muss zwischen 3 und 50 Zeichen lang sein. Beachten Sie, dass *documents.azure.com* an den gewählten Namen des Endpunkts angehängt wird, um den Endpunkt des DocumentDB-Kontos zu erstellen.

	- Wählen Sie unter **Abonnement** das Azure-Abonnement aus, das Sie mit dem DocumentDB-Konto verwenden möchten. Wenn Ihr Konto nur über ein Abonnement verfügt, wird dieses Konto standardmäßig ausgewählt.

	- Unter **Ressourcengruppe** können Sie eine Ressourcengruppe für Ihr DocumentDB-Konto auswählen oder erstellen. Standardmäßig wird eine vorhandene Ressourcengruppe im Azure-Abonnement ausgewählt. Sie können jedoch eine neue Ressourcengruppe erstellen, zu der Sie Ihr DocumentDB-Konto hinzufügen möchten. Weitere Informationen finden Sie unter [Verwenden des Azure-Portals zum Verwalten Ihrer Azure-Ressourcen](resource-group-portal.md).

	- Unter **Standort** können Sie einen geografischen Standort angeben, an dem Ihr DocumentDB-Konto gehostet wird.

4.	Klicken Sie nach der Konfiguration der Optionen für das neue DocumentDB-Konto auf **Erstellen**. Die Erstellung des DocumentDB-Kontos kann einige Minuten dauern. Sie können den Fortschritt im Startmenü überwachen, um den Status zu überprüfen.
![Screenshot der Kachel "Erstellen" im Startmenü – Online-Datenbankersteller](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-3.png)

	Sie können den Status auch über den Hub "Benachrichtigungen" überwachen.

	![Datenbanken schnell erstellen - Screenshot des Benachrichtigungs-Hubs, der zeigt, dass das DocumentDB-Konto erstellt wird](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-4.png)

	![Screenshot des Benachrichtigungs-Hubs, der zeigt, dass das DocumentDB-Konto erfolgreich erstellt und in einer Ressourcengruppe bereitgestellt wurde – Benachrichtigung für Online-Datenbankersteller](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-5.png)

5.	Nachdem das DocumentDB-Konto erstellt wurde, kann es mit den Standardeinstellungen im Onlineportal verwendet werden. Beachten Sie, dass die Standardkonsistenz des DocumentDB-Kontos auf **Sitzung** eingestellt ist. Sie können die Standardkonsistenzeinstellung anpassen. Klicken Sie dazu auf der Befehlsleiste oben auf das Symbol **Einstellungen** und anschließend auf dem Blatt **Alle Einstellungen** unter **Feature** auf **Standardkonsistenz**.

    ![Screenshot des Blatts "Ressourcengruppe" – Anwendungsentwicklung beginnen](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-6.png)

    ![Screenshot des Blatts „Konsistenzebene“ – Sitzungskonsistenz](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-7.png)

[How to: Create a DocumentDB account]: #Howto
[Next steps]: #NextSteps
[documentdb-manage]: ../articles/documentdb/documentdb-manage.md

<!---HONumber=AcomDC_0302_2016-->