---
title: "Sistema de Reservas de Voos com Ruby on Rails ✈️"
date: 2024-09-13
categories: [Blogging, Tutorial]
tag: [ruby, rails, forms]
---

## Introdução 
Recentemente, desenvolvi um sistema de reservas de voos utilizando Ruby on Rails, onde os usuários podem pesquisar voos, selecionar o voo desejado e inserir suas informações pessoais para concluir a reserva. Este post tem como objetivo compartilhar os detalhes do desenvolvimento dessa aplicação.

## Funcionalidades da Aplicação
### 1. Pesquisa de Voos
Na tela de pesquisa, o usuário pode selecionar o aeroporto de partida, aeroporto de chegada, data do voo e o número de passageiros. As opções de aeroportos são carregadas dinamicamente a partir do banco de dados, e o menu suspenso de datas mostra apenas os dias com voos disponíveis.

```ruby
class FlightsController < ApplicationController
  def index
    @airports = Airport.all
    @dates = Flight.pluck(:start_datetime).map { |d| d.to_date }.uniq
    @flights = []

    if params[:departure_airport_id].present? && params[:arrival_airport_id].present? && params[:date].present?
      @flights = Flight.where(
        departure_airport_id: params[:departure_airport_id],
        arrival_airport_id: params[:arrival_airport_id],
        start_datetime: params[:date].to_date.all_day
      )
    end
  end
end
```
### 2. Escolha do Voo
Após a pesquisa, os voos disponíveis são apresentados ao usuário, que pode selecionar o voo desejado. O formulário de seleção inclui os detalhes do voo, como horário de partida, chegada e duração.

```erb
<%= form_with url: new_booking_path, method: :get, local: true do %>
  <table>
    <thead>
      <tr>
        <th></th>
        <th>Aeroporto de Partida</th>
        <th>Aeroporto de Chegada</th>
        <th>Data e Hora</th>
        <th>Duração</th>
      </tr>
    </thead>
    <tbody>
      <% @flights.each do |flight| %>
        <tr>
          <td><%= radio_button_tag :flight_id, flight.id %></td>
          <td><%= flight.departure_airport.code %></td>
          <td><%= flight.arrival_airport.code %></td>
          <td><%= flight.start_datetime %></td>
          <td><%= flight.duration %> minutos</td>
        </tr>
      <% end %>
    </tbody>
  </table>
  <%= hidden_field_tag :passengers, params[:passengers] %>
  <%= submit_tag "Escolher Voo" %>
<% end %>
```
### 3. Captura das Informações dos Passageiros
Após selecionar o voo, o usuário insere as informações pessoais de cada passageiro. Cada passageiro recebe um campo de nome e e-mail, e as informações são enviadas para criar a reserva.
```ruby
class BookingsController < ApplicationController
  def new
    @flight = Flight.find(params[:flight_id])
    @booking = Booking.new(flight: @flight, passengers_count: params[:passengers])
    params[:passengers].to_i.times { @booking.passengers.build }
  end

  def create
    @booking = Booking.new(booking_params)
    if @booking.save
      redirect_to @booking, notice: 'Reserva criada com sucesso!'
    else
      render :new
    end
  end

  private

  def booking_params
    params.require(:booking).permit(:flight_id, :passengers_count, passengers_attributes: [:name, :email])
  end
end
```

### 4. Exibição da Reserva
Após o envio do formulário, a página de confirmação exibe as informações do voo e dos passageiros. O sistema associa automaticamente os passageiros à reserva e ao voo escolhido.
```erb
<h1>Reserva Confirmada</h1>
<p><strong>Aeroporto de Partida:</strong> <%= @booking.flight.departure_airport.code %></p>
<p><strong>Aeroporto de Chegada:</strong> <%= @booking.flight.arrival_airport.code %></p>
<p><strong>Data e Hora:</strong> <%= @booking.flight.start_datetime %></p>
<p><strong>Duração:</strong> <%= @booking.flight.duration %> minutos</p>

<h2>Passageiros</h2>
<ul>
  <% @booking.passengers.each do |passenger| %>
    <li>
      <p><strong>Nome:</strong> <%= passenger.name %></p>
      <p><strong>Email:</strong> <%= passenger.email %></p>
    </li>
  <% end %>
</ul>
```
## Tecnologias Utilizadas
* **Ruby on Rails:** Utilizado para construir o backend da aplicação, com ActiveRecord para gerenciar as associações entre voos, reservas e passageiros.
* **HTML e ERB:** Para construir as páginas de visualização com os formulários e exibir os dados.
* **PostgreSQL:** Banco de dados para armazenar os voos, reservas e informações dos passageiros.

## Desafios e Aprendizado
Durante o desenvolvimento, o principal desafio foi lidar com os parâmetros aninhados para criar múltiplos passageiros a partir de um único formulário de reserva. A solução envolveu o uso do método `accepts_nested_attributes_for` no modelo `Booking` para lidar com a criação em massa de passageiros.

## Conclusão
Este projeto me permitiu aprofundar meus conhecimentos em associações de modelos no Rails, uso de atributos aninhados e manipulação de formulários dinâmicos. Além disso, pude reforçar o uso de boas práticas no desenvolvimento web.[confira o codigo completo](https://github.com/fernandodxx/Flight-Booker)

