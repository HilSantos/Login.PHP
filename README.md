# Login.PHP
Criação do login.php

<?php
// Inclui arquivos de segurança, conexão com o banco de dados e mensagens de usuário //
session_start(); // Inicia a sessão para armazenar dados do usuário //
include('segurancazero.php');
include('conn.php');

// Verifica se o formulário foi enviado via POST //
if($_SERVER['REQUEST_METHOD']=='POST'){
    $email = $_POST['email'];
    $senha = $_POST['senha'];

    // Verifica se o e-mail existe no banco de dados //
    if(empty($email) || empty($senha)){
        // E-mail ou senha vazios: redireciona com mensagem de erro //
        header('Location: login.php?msg=Usuario e/ou senha invalidos!');
        exit();
    }
    // Sanitiza o e-mail para evitar injeção de SQL //
    $sql = "SELECT COUNT(*) FROM tb_usuarios WHERE email_usuario = '$email'";
    $result = mysqli_query($link, $sql);
    $contador = mysqli_fetch_array($result);

    if($contador[0] == 0){
        // E-mail não encontrado: redireciona com mensagem de erro //
        mysqli_close($link); // Fecha a conexão com o banco de dados //
        header('Location: login.php?msg=Usuario e/ou senha invalidos!');
        exit();
    } else {
        // E-mail encontrado: busca o tempero (salt) para validar a senha //
        // O tempero é uma string única para cada usuário, usada para aumentar a segurança do hash da senha //
        $sql = "SELECT tempero_usuario FROM tb_usuarios WHERE email_usuario = '$email'";
        $result = mysqli_query($link, $sql);
        $tbl = mysqli_fetch_array($result);
        $tempero = $tbl[0];
    }

    // Aplica hash MD5 à senha combinada com o tempero //
    $senha = md5($senha . $tempero);

    // Verifica se a senha está correta //
    $sql = "SELECT COUNT(*) FROM tb_usuarios WHERE email_usuario = '$email' AND senha_usuario = '$senha'";
    $result = mysqli_query($link, $sql);
    $count = mysqli_fetch_array($result);

    if($count[0] == 1){
        // Login válido: busca dados do usuário //
        $sql = "SELECT id_usuario, apelido_usuario, nivel_usuario
                FROM tb_usuarios 
                WHERE email_usuario = '$email' AND senha_usuario = '$senha'";
        echo $sql; // (provavelmente para debug, pode ser removido em produção) //
        $result = mysqli_query($link, $sql);
        $tbl = mysqli_fetch_array($result);

        // Armazena dados do usuário na sessão //
        $_SESSION['id_usuario'] = $tbl[0];
        $_SESSION['apelido'] = $tbl[1];
        $_SESSION['nivel'] = $tbl[2];

        mysqli_close($link);
        header('Location: index.php'); // Redireciona para a página inicial //
        exit();
    } else {
        // Senha incorreta //
        mysqli_close($link);
        header('Location: login.php?msg=Usuario e/ou senha invalidos!');
        exit();
    }
}
?>


<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="lista.css">
    <title>Login</title>
</head>
<body>
    <br>
    <h1>Login</h1>
    <!-- Exibe mensagens de erro, se houver -->
    <?php
    include('msg_user.php');
    ?>
    <br><br>
    <!-- Formulário de login -->
    <form action="login.php" method="post">
        <label for="email">Email</label>
        <input type="email" name="email" id="email" required>
        <br>
        <label for="senha">Senha</label>
        <input type="password" name="senha" id="senha" required>
        <br><br>
        <input type="submit" value="Entrar">
    </form>
    <!-- Link para redefinir senha, exibido apenas se houver erro de login -->
    <?php
    if(isset($_GET['msg']) && $_GET['msg'] == 'Usuario e/ou senha invalidos!'){
        ?>
    <br>
    <a href="recuperasenha.php">Redefinir Senha</a>
    <?php
    }
    ?>
</body>
</html>
