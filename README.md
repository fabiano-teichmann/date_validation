# Teste unitários para validação e conversão de datas
Esse projeto visa de maneira prática como implementar testes unitários para isso utilizei a biblioteca padrão do python o unittest. Essa demanda veio de um projeto onde eu trabalharia com a integração de dados de diversas fontes e de diversos formatos como americano com minutos e segundos, o formato brasileiro somente com data. Para garantir a integridade e interoperabilidade entre os sistema era necessário que a minha implementação converte-se datas de um formato específico para datetime e de datetime para outro padrão. E com a filosofia Test Fist comecei meu trabalho importando as bibliotecas necessárias.


    from unittest import TestCase
    from datetime import datetime
    from utils import ConvertDate

Depois comecei escrevendo minha classe e começo com a função default do unittest o setUp onde crio as listas que serão usandas para 
testar meu código.

    class TestConvertDate(TestCase):
        def setUp(self):
        self.dates = ['02-12-2018', '01/12/2018 00:31:29', 01-10-2018 00:31:29', '02/12/2018', '02-12-2018']
        self.dates_invalid = ['catota no nariz', '00:01:01 2018/01/20', '31/02/2018', 1, dict(ola='oi'), '31\02/2018', '&02#2018']

Para começar a brincadeira eu começo com  duas lista um de datas válidas de diversos formatos e padrões  e outra de datas inválidas.
O proposito dessas datas invalidas é forçar uma quebra no programa, quando não consigo converter o comportamento desejado pelo meu 
programa é retornar um dicionário com um código de resposta padrão do protocolo http e uma mensagem, por isso pensei nas mais loucas 
possibilidades de usuário colocar algo invalido na data, como por exemplo catota no nariz kkkk. 

## Testando converte uma data no formato brasileiro para datetime
Eu pego minha lista de datas no formato brasileiro válido e testo se minha função converte para datetime.
O propósito deste teste é ter certeza que daqui um tempo quando adicionar novos formato de datas que as pessoas vão colocar eu possa verificar de maneira simples e prática se seu software ainda consegue converter para datetime.
E para finalizar tenho ter certeza que se eu converter de datetime para string no formato brasileiro o input seja no formato "dia/mês/ano".
E que seja igual a data de entrada.


        def test_convert_string_brazil_to_datetime(self):
        """Test convert datetime for string in format date brazil"""

        for d in self.dates_brasil:
            date = ConvertDate().convert_date_brasil_to_datetime(d)
            self.assertIsInstance(date, datetime, msg="A função não consegui converter para datetime")
            date_brasil = ConvertDate().convert_datetime_to_date_brazil(date)
            """Aqui faço a troca do - pelo / pois o usuário pode digitar com o -  e removo a hora minutos e segundo
            """
            self.assertEqual(date_brasil,  d[0:10].replace('-', '/'),
                             msg="Opa o output deve ser igual ao input")

Somente agora com esse cenário definido do que o que eu espero em caso de sucesso e em caso de falha eu começo escrever meu código em si


    def convert_date_brasil_to_datetime(self, date):
        """
        Receive string date in format brazil and and convert in datetime

        Args:
            date (str): date

        Returns:
            datetime in format year/month/day hours:minute:seconds or return dict with  code error
        """
        date = self.valid_string(date)
        if isinstance(date, dict):
            return date
        try:
            date = date.replace('-', '/')
            return datetime.strptime(date, self.datetime_brazil)
        except ValueError:
            return dict(code=415, help=self.format_invalid, msg='Essa data não é válida tem  dígitos sobrando')
        except Exception as e:
            raise e

Começo  chamando a função valid_string() que recebe a data em formato string e tenta verificar se a string pode ser convertida para data.

    def valid_string(self, date):
        """
        Validator if string can convert for datetime. The function try format string for format valid if not possible
        return dict with code 415, msg and help
        Args:
            date (str): date to be validate

        Returns:
            string or dict
        """
        if isinstance(date, str):
            if len(date) == 8:
                return dict(code=415, help=self.format_invalid, msg='A data tem que ter 8 digitos')

            elif len(date) == 10:
                date = f'{date} 00:00:00'
                if date.count('/') == 2 or date.count('-') == 2 and date.count(':') == 2:
                    return date
                else:
                    return dict(code=415, help=self.format_invalid, msg='Está faltando as duas / na data')
            elif len(date) == 19:
                return date
            elif len(date) > 19:
                return dict(code=415, help=self.format_invalid, msg='Essa data não é válida tem numeros sobrando')
            else:
                return dict(code=415, help=self.format_invalid, msg='Essa data não é válida')
        else:
            return dict(code=415, help=self.format_invalid, msg='Opa você está usando caracteres não válidos')


Como não posso ter certeza que o usuário vai digitar a data num padrão que desejo tento prever alguns padrões de datas e adiciono a hora minutos e segundos, para que minha função consiga converter para datetime e que eu possa salvar no meu banco de dados no padrão datetime.

## Testando a função que converte data e hora no padrão americano para datetime
Uma api que consulto traz a data nesse formato "2018-12-31 04:32:32". 
Então implemento teste para validar se meu código está conseguindo converter para datetime assim como converti datas
no padrão brasileiro para datetime.

        def test_convert_to_string_american_to_datetime(self):
        """Test convert datetime for string in format american '%Y-%m-%d %H:%M:%S'"""
        for d in self.date_american:
            date = ConvertDate().convert_date_american_to_datetime(d)
            self.assertIsInstance(date, datetime, msg="A função não consegui converter para datetime")
         
Com o teste criado começo desenvolver minha função para converter data e hora no formato americano para datetime

        def convert_date_american_to_datetime(self, date):
        """
        Receive string date in format american and and convert in datetime

        Args:
            date (str): date

        Returns:
            datetime in format year/month/day hours:minute:seconds or return dict with  code error
        """
        date = self.valid_string(date)
        if isinstance(date, dict):
            return date
        try:
            date = date.replace('/', '-')
            return datetime.strptime(date, self.datetime_american)
        except ValueError:
            return dict(code=415, help=self.format_invalid, msg='Essa data não é válida tem  dígitos sobrando')
        except Exception as e:
            raise e
            
Minha função pode retornar um exceção mas esse não é o que espero, o comportamento que desejo é que retorne uma mensagem com código de erro.
  Espero que com isso ajude a esclarecer sobre testes unitários e perguntas e criticas são bem vindo.
            
             

