import SwiftUI

// MARK: - Modelo de dados da API
struct Pokemon: Identifiable, Decodable {
    let id = UUID()
    let name: String
    let url: String
    
    var imageURL: String {
        let id = url.split(separator: "/").dropLast().last ?? "1"
        return "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/\(id).png"
    }
}

struct PokemonResponse: Decodable {
    let results: [Pokemon]
}

// MARK: - ViewModel
@MainActor
class PokedexViewModel: ObservableObject {
    @Published var pokemons: [Pokemon] = []
    @Published var isLoading = false
    
    func fetchPokemons() async {
        guard let url = URL(string: "https://pokeapi.co/api/v2/pokemon?limit=151") else { return }
        
        isLoading = true
        defer { isLoading = false }
        
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let decoded = try JSONDecoder().decode(PokemonResponse.self, from: data)
            pokemons = decoded.results
        } catch {
            print("Erro ao buscar Pokémons:", error)
        }
    }
}

// MARK: - Tela principal
struct ContentView: View {
    @StateObject private var viewModel = PokedexViewModel()
    
    var body: some View {
        NavigationView {
            Group {
                if viewModel.isLoading {
                    ProgressView("Carregando Pokédex...")
                } else {
                    List(viewModel.pokemons) { pokemon in
                        NavigationLink(destination: PokemonDetailView(pokemon: pokemon)) {
                            HStack {
                                AsyncImage(url: URL(string: pokemon.imageURL)) { image in
                                    image
                                        .resizable()
                                        .scaledToFit()
                                        .frame(width: 50, height: 50)
                                } placeholder: {
                                    ProgressView()
                                }
                                Text(pokemon.name.capitalized)
                                    .font(.headline)
                            }
                        }
                    }
                }
            }
            .navigationTitle("Pokédex")
            .task {
                await viewModel.fetchPokemons()
            }
        }
    }
}

// MARK: - Tela de detalhes
struct PokemonDetailView: View {
    let pokemon: Pokemon
    
    var body: some View {
        VStack(spacing: 20) {
            AsyncImage(url: URL(string: pokemon.imageURL)) { image in
                image
                    .resizable()
                    .scaledToFit()
                    .frame(width: 200, height: 200)
            } placeholder: {
                ProgressView()
            }
            
            Text(pokemon.name.capitalized)
                .font(.largeTitle)
                .bold()
            
            Text("Mais detalhes em breve!")
                .foregroundColor(.gray)
        }
        .padding()
        .navigationTitle(pokemon.name.capitalized)
    }
}

// MARK: - App
@main
struct PokedexApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
