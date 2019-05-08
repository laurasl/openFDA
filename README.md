import json 
import socketserver
import http.server
import http.client
 

PORT = 8000  


class testHTTPRequestHandler(http.server.BaseHTTPRequestHandler):

    link_openfda = "api.fda.gov"
    json_openfda = "/drug/label.json"
    medicamento_openfda = '&search=active_ingredient:'
    compañia_openfda = '&search=openfda.manufacturer_name:'

    def pag_inicial(self):  
        # la estructura de nuestra pagina viene determinada por este html
        html = """   
            <html>
                <head>
                    <title> OpenFDA</title>
                </head>
                <body align=center>
                    <h1>Drug product labelling OpenFDA </h1>
                    <form method="get" action="listDrugs">
                        <input type = "submit" value="Drug List">
                        </input>
                    </form>
                    <br>
                    <br>
                    <form method="get" action="searchDrug">
                        <input type = "submit" value="Drug Search">
                        <input type = "text" name="drug"></input>
                        </input>
                    </form>
                    <br>
                    <br>
                    <form method="get" action="listCompanies">
                        <input type = "submit" value="Company List">
                        </input>
                    </form>
                    <br>
                    <br>
                    <form method="get" action="searchCompany">
                        <input type = "submit" value="Company Search">
                        <input type = "text" name="company"></input>
                        </input>
                    </form>
                    <br>
                    <br>
                    <form method="get" action="listWarnings">
                        <input type = "submit" value="Warnings List">
                        </input>
                    </form>
                </body>
            </html>
                """
        return html

    def segunda_pag(self, lista): 
        datos_html = """
                                <html>
                                    <head>
                                        <title>Laura´s App</title>   
                                    </head>
                                    <body style='background-color: lightpink'>
                                        <ul>
                            """
        for i in lista:  
            datos_html += "<li>" + i + "</li>"

        datos_html += """
                                        </ul>
                                    </body>
                                </html>
                            """
        return datos_html

    def results_obtenidos(self, limit=10):  # conseguimos unos resultados
        conectar = http.client.HTTPSConnection(self.link_openfda)  # establecemos conexion
        conectar.request("GET", self.json_openfda + "?limit=" + str(limit))  
        print(self.json_openfda + "?limit=" + str(limit))
        resp_1 = conectar.getresponse()  # obtenemos la respuesta por parte del servidor
        datos_cod = resp_1.read().decode("utf8")  # Leer el contenido en json, y transformarlo en una cadena
        datos = json.loads(datos_cod)  # procesamos el contenido json
        results_obtenidos = datos['results']
        return results_obtenidos

    def do_GET(self):
        list_medios = self.path.split("?")
        if len(list_medios) > 1:
            parametros = list_medios[1]
        else:
            parametros = ""

        limit = 1

        if parametros:
            fragmento = parametros.split("=")
            if fragmento[0] == "limit":
                limit = int(fragmento[1])
                print("Limit: {}".format(limit))
        else:
            print("SIN PARAMETROS")

        if self.path == '/':

            self.send_response(200)  
            self.send_header('Content-type', 'text/html') 
            self.end_headers()
            html = self.pagina_inicio()
            self.wfile.write(bytes(html, "utf8"))

        elif 'listDrugs' in self.path:  
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            medicamento = []
            results_obtenidos = self.results_obtenidos(limit)
            for resultado in results_obtenidos:
                if ('generic_name' in resultado['openfda']):
                    medicamento.append(resultado['openfda']['generic_name'][0])
                else:
                    medicamento.append('Desconocido')
            resultado_html = self.pagina_2(medicamento)

            self.wfile.write(bytes(resultado_html, "utf8"))

        elif 'listCompanies' in self.path:  
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            compañia = []
            results_obtenidos = self.results_obtenidos(limit)
            for resultado in results_obtenidos:
                if ('manufacturer_name' in resultado['openfda']):
                    compañia.append(resultado['openfda']['manufacturer_name'][0])
                else:
                    compañia.append('Desconocido')
            resultado_html = self.pagina_2(compañia)

            self.wfile.write(bytes(resultado_html, "utf8"))
        elif 'listWarnings' in self.path:  

            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            advs = []
            results_obtenidos = self.results_obtenidos(limit)
            for resultado in results_obtenidos:  # introducimos nuestros results_obtenidos en una lista
                if ('advs' in resultado):
                    advs.append(resultado['advs'][0])
                else:
                    advs.append('Desconocido')
            resultado_html = self.pagina_2(advs)

            self.wfile.write(bytes(resultado_html, "utf8"))

        elif 'searchDrug' in self.path:  

            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            
            limit = 10
            drug = self.path.split('=')[1]

            medicamentos = []
            conectar = http.client.HTTPSConnection(self.link_openfda) 
            conectar.request("GET", self.json_openfda + "?limit=" + str(limit) + self.medicamento_openfda + drug)
            resp_1 = conectar.getresponse()
            datos1 = resp_1.read()
            datos = datos1.decode("utf8")
            biblioteca_datos = json.loads(datos)
            events_search_drug = biblioteca_datos['results']
            for resultado in events_search_drug:
                if ('generic_name' in resultado['openfda']):
                    medicamentos.append(resultado['openfda']['generic_name'][0])
                else:
                    medicamentos.append('Desconocido')

            resultado_html = self.pagina_2(medicamentos)
            self.wfile.write(bytes(resultado_html, "utf8"))


        elif 'searchCompany' in self.path: 

            self.send_response(200) 
            self.send_header('Content-type', 'text/html')
            self.end_headers()

            limit = 10
            company = self.path.split('=')[1]
            compañias = []
            conectar = http.client.HTTPSConnection(self.link_openfda)
            conectar.request("GET", self.json_openfda + "?limit=" + str(limit) + self.compañia_openfda + company)
            resp_1 = conectar.getresponse()
            datos1 = resp_1.read()
            datos = datos1.decode("utf8")
            biblioteca_company = json.loads(datos)
            search_company = biblioteca_company['results']

            for search in search_company:
                compañias.append(search['openfda']['manufacturer_name'][0])
            resultado_html = self.pagina_2(compañia)
            self.wfile.write(bytes(resultado_html, "utf8"))

        # Aquí tenemos una serie de extensiones

        elif 'redirect' in self.path:
            print('Volvemos a la página principal')
            self.send_response(301)
            self.send_header('Location', 'http://localhost:' + str(PORT))
            self.end_headers()

        elif 'secret' in self.path:
            self.send_response(401)
            self.send_header('WWW-Authenticate', 'Basic realm="Mi servidor"')
            self.end_headers()

        else:  # Si el recurso solicitado no se encuentra en el servidor.
            self.send_error(404)
            self.send_header('Content-type', 'text/plain; charset=utf-8')
            self.end_headers()
            self.wfile.write("ERROR 404, NOT FOUND '{}'.".format(self.path).encode())
        return


socketserver.TCPServer.allow_reuse_address = True  
Handler = testHTTPRequestHandler
httpd = socketserver.TCPServer(("", PORT), Handler)
print("serving at port", PORT)

httpd.serve_forever()
