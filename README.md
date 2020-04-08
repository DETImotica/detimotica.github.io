# Website do Projeto DETImótica
**[detimotica.github.io](https://detimotica.github.io/)**

## Instalação Local
1. Clonar o repositório
```
git clone https://github.com/DETImotica/detimotica.github.io.git
```

#### Windows 10 Nativo ou WSL

Por favor seguir as instruções da seguinte página: [jekyllrb.com/docs/installation/windows/](https://jekyllrb.com/docs/installation/windows/)

#### Linux

2. Instalar as dependências de sistema
```
sudo apt-get install ruby-full build-essential zlib1g-dev
```

3. Permitir a execução do Ruby Gems sem permissões de administração (Opcional)
```
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

4. Navegar até ao diretório do repositório
```
cd detimotica.github.io
```

5. Instalar as dependências do website
```
gem install bundler
bundle install
```

<br>
## Utilização

No diretório do projeto, executar:
```
bundle exec jekyll serve
```

O website poderá ser acedido no endereço [127.0.0.1:4000](http://127.0.0.1:4000).

As alterações ao código serão automaticamente compiladas aquando da gravação dos ficheiros, pelo que bastará atualizar a página para que as mesmas se reflitam.

### Markdown (Kramdown)

Para referências sobre a elaboração de páginas em Markdown e ferramentas de ajuda por favor consultar:
- [www.markdownguide.org/cheat-sheet/](https://www.markdownguide.org/cheat-sheet/)
- [www.tablesgenerator.com/markdown_tables](https://www.tablesgenerator.com/markdown_tables)

Template: Edition
